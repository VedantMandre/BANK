
1. The Core Engine (ProfileAccessor.java)
This class now only handles the "Infrastructure"—connecting to the DB and executing a generic task. It doesn't need to know what a "Zachrtn" is.

```
Java
package com.smbcgroup.cms.reference;

import java.sql.*;
import java.util.*;
import java.util.logging.Logger;

public class ProfileAccessor {
    private static final Logger LOGGER = Logger.getLogger(ProfileAccessor.class.getName());
    
    // Infrastructure variables
    private final String hostname = System.getenv("HOST_NAME");
    private final String username = System.getenv("CONNECTION_USERNAME");
    private final String password = System.getenv("CONNECTION_PASSWORD");

    /**
     * The ONLY method needed in ProfileAccessor for all future tables.
     * It accepts a SQL query and a way to map the results.
     */
    public <T> List<T> executeAndMap(String query, RowMapper<T> mapper) throws Exception {
        List<T> results = new ArrayList<>();
        Class.forName("sanchez.jdbc.driver.scDriver");

        try (Connection conn = DriverManager.getConnection(hostname, username, password);
             PreparedStatement stmt = conn.prepareStatement(query);
             ResultSet rs = stmt.executeQuery()) {
            
            while (rs.next()) {
                results.add(mapper.map(rs));
            }
        } catch (Exception e) {
            LOGGER.severe("Database execution failed: " + e.getMessage());
            throw e;
        }
        return results;
    }
}

@FunctionalInterface
interface RowMapper<T> {
    T map(ResultSet rs) throws SQLException;
}
```
2. The Smart Model (Zachrtn.java)
We put the "Knowledge" of the table inside the model. By creating a static method here, you keep all logic related to ZACHRTN in one file.
```
Java
package com.smbcgroup.cms.reference;

import lombok.*;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Zachrtn {
    // 1. Define Columns as a Constant for easy maintenance
    public static final String SELECT_QUERY = 
        "SELECT TJD, SRC, CID, SPR, ZRTP, ZTCD, ZRDI, ZCD, ZDCID, ZAMNT, ZID, ZIDNM, " +
        "ZDISCD, ZADDEND, ZTRACE, ZDATE, ZCLS, ZENTRY, ZORDI, ZCOID, ZCONAM, IAT, IATPOFAC, IATSOFAC " +
        "FROM ZACHRTN";

    // Fields
    private String tjd; 
    private String src, cid, spr, zrtp, ztcd, zrdi, zcd, zdcid, zamnt, zid, zidnm;
    private String zdiscd, zaddend, ztrace, zdate, zcls, zentry, zordi, zcoid, zconam;
    private String iat, iatpofac, iatsofac;

    /**
     * 2. The 1-1 Column Mapper logic lives here.
     */
    public static Zachrtn fromResultSet(ResultSet rs) throws SQLException {
        return Zachrtn.builder()
                .tjd(rs.getString("TJD"))
                .src(rs.getString("SRC"))
                .cid(rs.getString("CID"))
                .spr(rs.getString("SPR"))
                .zrtp(rs.getString("ZRTP"))
                .ztcd(rs.getString("ZTCD"))
                // ... add all other fields using rs.getString("COLUMN_NAME") ...
                .iatsofac(rs.getString("IATSOFAC"))
                .build();
    }

    /**
     * 3. The service method you call from the Function.
     */
    public static List<Zachrtn> fetchAll(ProfileAccessor accessor) throws Exception {
        return accessor.executeAndMap(SELECT_QUERY, Zachrtn::fromResultSet);
    }
}
```
3. The Clean Function Call (Function.java)
Now, your Azure Function doesn't care about SQL or mapping. It just asks the Model to fetch itself.
```
Java
@FunctionName("RetrieveZachrtnData")
public HttpResponseMessage fetchZachrtn(
        @HttpTrigger(name = "req", methods = {HttpMethod.GET}, authLevel = AuthorizationLevel.ANONYMOUS) 
        HttpRequestMessage<Optional<String>> request,
        final ExecutionContext context) {
    
    try {
        // Just one line to get data:
        List<Zachrtn> data = Zachrtn.fetchAll(this.profileAccessor);
        
        return getHttpResponseMessage(request, data);
    } catch (Exception e) {
        context.getLogger().severe("Error: " + e.getMessage());
        return request.createResponseBuilder(HttpStatus.INTERNAL_SERVER_ERROR).build();
    }
}
```
Why this solves your problem:
Isolation: If you add a new table (e.g., Invoices), you create Invoice.java, add the SQL and Mapper there, and ProfileAccessor.java never changes.

No If-Else: The generic executeAndMap handles everything. The "Mapping" logic is passed as a reference (Zachrtn::fromResultSet).

Readability: ProfileAccessor stays short (about 30-40 lines) forever.

Rigid Mapping: You are using rs.getString("COLUMN_NAME") inside the model, ensuring the data always goes to the right field.


