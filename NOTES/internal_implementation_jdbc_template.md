# Internal Implementation of `JdbcTemplate`

```java
class JdbcTemplate {
    private DataSource dataSource;

    public JdbcTemplate() {}
    public JdbcTemplate(DataSource dataSource) {}
    public void setDataSource(DataSource dataSource) {}

    public Object queryForObject(String sql, Class<?> classType) {
        Connection con = null;
        Statement stmt = null;
        ResultSet rs = null;
        Object result = null;
        try {
            con = dataSource.getConnection();
            stmt = con.createStatement();
            rs = stmt.executeQuery(sql);

            if (rs.next()) {
                if (classType == Integer.class) {
                    result = rs.getInt(1);
                } else if (classType == Float.class) {
                    result = rs.getFloat(1);
                } else if (classType == String.class) {
                    result = rs.getString(1);
                }
            }
        } catch (SqlException e) {
            throw new DataAccessException(e); // unchecked spring jdbc exception
        }
        return result;
    }

    public Object queryForObject(String sql, Class<?> classType, Object[] params) {
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        Object result = null;
        try {
            con = dataSource.getConnection();
            pstmt = con.preparedStatement(sql);
            // substitute the parameters based on positions
            rs = pstmt.executeQuery();

            if (rs.next()) {
                if (classType == Integer.class) {
                    result = rs.getInt(1);
                } else if (classType == Float.class) {
                    result = rs.getFloat(1);
                } else if (classType == String.class) {
                    result = rs.getString(1);
                }
            }
        } catch (SqlException e) {
            throw new DataAccessException(e); // unchecked spring jdbc exception
        }
        return result;
    }
}
````
