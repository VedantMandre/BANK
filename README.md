```
package com.smbcgroup.cms.reference;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Zachrtn {
    private String tjd;         // Julian Time
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
```


```
import java.sql.ResultSet;
import java.sql.SQLException;

// Functional interface to handle mapping
@FunctionalInterface
interface RowMapper<T> {
    T map(ResultSet rs) throws SQLException;
}

public class ProfileAccessor {
    // ... (Existing variables and methods) ...

    /**
     * The Modular Engine
     * T is the type of object (e.g., Zachrtn)
     */
    private <T> List<T> executeAndMap(String query, RowMapper<T> mapper) throws Exception {
        List<T> results = new ArrayList<>();
        Class.forName("sanchez.jdbc.driver.scDriver");

        try (Connection conn = getConnection(hostname, connectionUsername, connectionPassword);
             PreparedStatement stmt = conn.prepareStatement(query);
             ResultSet rs = stmt.executeQuery()) {
            
            while (rs.next()) {
                results.add(mapper.map(rs));
            }
        } catch (Exception e) {
            LOGGER.severe("Query Failed: " + e.getMessage());
            throw e;
        }
        return results;
    }

    /**
     * Specific implementation for ZACHRTN
     * Uses column names for 1-1 matching to prevent index mismatches
     */
    public List<Zachrtn> getZachrtnData() throws Exception {
        String query = "SELECT TJD, SRC, CID, SPR, ZRTP, ZTCD, ZRDI, ZCD, ZDCID, ZAMNT, ZID, ZIDNM, " +
                       "ZDISCD, ZADDEND, ZTRACE, ZDATE, ZCLS, ZENTRY, ZORDI, ZCOID, ZCONAM, " +
                       "IAT, IATPOFAC, IATSOFAC FROM ZACHRTN";

        return executeAndMap(query, rs -> Zachrtn.builder()
                .tjd(rs.getString("TJD"))
                .src(rs.getString("SRC"))
                .cid(rs.getString("CID"))
                .spr(rs.getString("SPR"))
                .zrtp(rs.getString("ZRTP"))
                .ztcd(rs.getString("ZTCD"))
                .zrdi(rs.getString("ZRDI"))
                .zcd(rs.getString("ZCD"))
                .zdcid(rs.getString("ZDCID"))
                .zamnt(rs.getString("ZAMNT"))
                .zid(rs.getString("ZID"))
                .zidnm(rs.getString("ZIDNM"))
                .zdiscd(rs.getString("ZDISCD"))
                .zaddend(rs.getString("ZADDEND"))
                .ztrace(rs.getString("ZTRACE"))
                .zdate(rs.getString("ZDATE"))
                .zcls(rs.getString("ZCLS"))
                .zentry(rs.getString("ZENTRY"))
                .zordi(rs.getString("ZORDI"))
                .zcoid(rs.getString("ZCOID"))
                .zconam(rs.getString("ZCONAM"))
                .iat(rs.getString("IAT"))
                .iatpofac(rs.getString("IATPOFAC"))
                .iatsofac(rs.getString("IATSOFAC"))
                .build()
        );
    }
}
```
```
@FunctionName("RetrieveZachrtnData")
public HttpResponseMessage fetchZachrtn(
        @HttpTrigger(name = "req", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.ANONYMOUS) 
        HttpRequestMessage<Optional<String>> request,
        final ExecutionContext context) {
    
    try {
        List<Zachrtn> data = profileAccessor.getZachrtnData();
        return getHttpResponseMessage(request, data);
    } catch (Exception e) {
        context.getLogger().severe("Error: " + e.getMessage());
        return request.createResponseBuilder(HttpStatus.INTERNAL_SERVER_ERROR).build();
    }
}
```

