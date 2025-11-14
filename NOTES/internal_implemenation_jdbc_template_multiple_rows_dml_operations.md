# Internal Implementation for JdbcTemplate with Multiple Rows + DML Operations

```java
class JdbcTemplate {

    private DataSource dataSource;

    public JdbcTemplate() {}
    public JdbcTemplate(DataSource dataSource) {}
    public void setDataSource(DataSource dataSource) {}

    public List<Object> query(String sql, RowMapper rowMapper) {
        ResultSet rs = null;
        Connection con = null;
        List objects = null;
        int rowNum = 0;
        PreparedStatement pstmt = null;

        try {
            con = dataSource.getConnection();
            pstmt = con.preparedStatement(sql);
            rs = pstmt.executeQuery();
            objects = new ArrayList();

            while (rs.next()) {
                object = rowMapper.mapRow(rs, rowNum);
                objects.add(object);
                rowNum++;
            }

        } catch (SqlException e) {
            throw new DataAccessException(e);
        }

        return objects;
    }

    public int update(String sql, Object[] params) {
        int r = 0;
        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = dataSource.getConnection();
            pstmt = con.preparedStatement(sql);
            // substitute parameters
            r = pstmt.executeUpdate();

        } catch (SqlException e) {
            throw new DataAccessException(e);
        }

        return r;
    }
}
````

---

### ➤ Note

**RowMapper** is a callback class invoked by `JdbcTemplate` to map each record into an object after executing the SQL query and while iterating over the `ResultSet`.

```
