What We’re Going To Build

A clean layered JDBC design:

Function Layer
    ↓
Service Layer
    ↓
Repository Layer (Generic)
    ↓
Query Enum
    ↓
RowMapper


So when a new table (like ZACHRTN) comes:

Add DTO

Add Query enum entry

Add RowMapper

Done ✅

NO touching core logic.

✅ STEP 1 — Move Queries to Enum (Very Good Idea)

Create:

ProfileQuery.java
package com.smbcgroup.cms.reference;

public enum ProfileQuery {

    ACCOUNTS(
        "SELECT a.GRP, a.CID, a.ACN, a.BOO, a.TITLE1, a.TITLE2, " +
        "a.ACCTNAME, a.TYPE, a.CRCD, a.STAT, " +
        "addr.AD1, addr.AD2, addr.AD3, addr.AD4 " +
        "FROM ACN a JOIN ACNADDR addr ON a.CID = addr.CID"
    ),

    COMPANIES(
        "SELECT ACN, BOO, NAM, PAD1, PAD2, PCITY, PZIP, PSTATE FROM CIF"
    ),

    ZACHRTN(
        "SELECT TJD, SRC, CID, SPR, ZRTP, ZTCD, ZRDI, ZCD, ZDCID, ZAMNT, ZID, ZIDNM, " +
        "ZDISCD, ZADDEND, ZTRACE, ZDATE, ZCLS, ZENTRY, ZORDI, ZCOID, ZCONAM, " +
        "IAT, IATPOFAC, IATSOFAC FROM ZACHRTN"
    );

    private final String sql;

    ProfileQuery(String sql) {
        this.sql = sql;
    }

    public String getSql() {
        return sql;
    }
}


🔥 Now queries are centralized.
No more string guessing.

✅ STEP 2 — Generic Repository (Core Engine)

Refactor ProfileAccessor into:

ProfileRepository.java
package com.smbcgroup.cms.reference;

import java.sql.*;
import java.util.*;
import java.util.logging.Logger;

public class ProfileRepository {

    private static final Logger LOGGER = Logger.getLogger(ProfileRepository.class.getName());

    private final String hostname = System.getenv("HOST_NAME");
    private final String username = System.getenv("CONNECTION_USERNAME");
    private final String password = System.getenv("CONNECTION_PASSWORD");

    static {
        try {
            Class.forName("sanchez.jdbc.driver.scDriver");
        } catch (ClassNotFoundException e) {
            throw new RuntimeException("Driver not found", e);
        }
    }

    public <T> List<T> execute(ProfileQuery query,
                               RowMapper<T> mapper,
                               Object... params) throws Exception {

        List<T> results = new ArrayList<>();

        try (Connection connection = DriverManager.getConnection(hostname, username, password);
             PreparedStatement statement = connection.prepareStatement(query.getSql())) {

            // Handle dynamic parameters
            for (int i = 0; i < params.length; i++) {
                statement.setObject(i + 1, params[i]);
            }

            try (ResultSet rs = statement.executeQuery()) {
                while (rs.next()) {
                    results.add(mapper.mapRow(rs));
                }
            }

        } catch (Exception e) {
            LOGGER.severe("Database Error: " + e.getMessage());
            throw e;
        }

        return results;
    }
}


🔥 Now we have:

Generic execution

Parameter support

Enum-based queries

Clean separation

✅ STEP 3 — Fix Column Mapping (No Index Based Mapping)

Now create a mapper for each DTO.

AccountMapper.java
public class AccountMapper implements RowMapper<Account> {

    @Override
    public Account mapRow(ResultSet rs) throws SQLException {
        return new Account(
            rs.getString("GRP"),
            rs.getString("CID"),
            rs.getString("ACN"),
            rs.getString("BOO"),
            rs.getString("TITLE1"),
            rs.getString("TITLE2"),
            rs.getString("ACCTNAME"),
            rs.getString("TYPE"),
            rs.getString("CRCD"),
            rs.getString("STAT"),
            rs.getString("AD1"),
            rs.getString("AD2"),
            rs.getString("AD3"),
            rs.getString("AD4")
        );
    }
}


✔ No index
✔ Safe
✔ Order independent

✅ STEP 4 — Add ZACHRTN Cleanly

Create DTO:

Zachrtn.java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Zachrtn {

    private String tjd;
    private String src;
    private String cid;
    private String spr;
    private String zrtp;
    private String ztcd;
    private String zrdi;
    private String zcd;
    private String zdcid;
    private String zamnt;
    private String zid;
    private String zidnm;
    private String zdiscd;
    private String zaddend;
    private String ztrace;
    private String zdate;
    private String zcls;
    private String zentry;
    private String zordi;
    private String zcoid;
    private String zconam;
    private String iat;
    private String iatpofac;
    private String iatsofac;
}


Create Mapper:

public class ZachrtnMapper implements RowMapper<Zachrtn> {

    @Override
    public Zachrtn mapRow(ResultSet rs) throws SQLException {
        return new Zachrtn(
            rs.getString("TJD"),
            rs.getString("SRC"),
            rs.getString("CID"),
            rs.getString("SPR"),
            rs.getString("ZRTP"),
            rs.getString("ZTCD"),
            rs.getString("ZRDI"),
            rs.getString("ZCD"),
            rs.getString("ZDCID"),
            rs.getString("ZAMNT"),
            rs.getString("ZID"),
            rs.getString("ZIDNM"),
            rs.getString("ZDISCD"),
            rs.getString("ZADDEND"),
            rs.getString("ZTRACE"),
            rs.getString("ZDATE"),
            rs.getString("ZCLS"),
            rs.getString("ZENTRY"),
            rs.getString("ZORDI"),
            rs.getString("ZCOID"),
            rs.getString("ZCONAM"),
            rs.getString("IAT"),
            rs.getString("IATPOFAC"),
            rs.getString("IATSOFAC")
        );
    }
}

✅ STEP 5 — Function Layer (Clean & Modular)

Now your Azure Functions become tiny and clean.

@FunctionName("RetrieveZachrtn")
public HttpResponseMessage getZachrtn(
        @HttpTrigger(name = "req",
                     methods = {HttpMethod.GET},
                     authLevel = AuthorizationLevel.FUNCTION)
        HttpRequestMessage<Optional<String>> request,
        ExecutionContext context) {

    try {
        ProfileRepository repo = new ProfileRepository();

        List<Zachrtn> result =
                repo.execute(ProfileQuery.ZACHRTN,
                             new ZachrtnMapper());

        return request.createResponseBuilder(HttpStatus.OK)
                .header("Content-Type", "application/json")
                .body(result)
                .build();

    } catch (Exception e) {
        context.getLogger().severe("Error fetching ZACHRTN: " + e.getMessage());
        return request.createResponseBuilder(HttpStatus.INTERNAL_SERVER_ERROR).build();
    }
}

🧠 What We Achieved
✅ ProfileAccessor no longer bloats
✅ No if-else ladder
✅ No query.contains
✅ No index-based column mapping
✅ Queries centralized
✅ Adding new table = DTO + Mapper + Enum entry
✅ Production-grade separation
