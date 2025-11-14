# Internal Implementation for RowMapper

```java
class JdbcTemplate {
    private DataSource dataSource;

    public JdbcTemplate() {}
    public JdbcTemplate(DataSource dataSource) {}
    public void setDataSource(DataSource dataSource) {}

    public Object queryForObject(String sql, RowMapper rowMapper, Object[] params) {
        Object obj = null;
        ResultSet rs = null;
        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = dataSource.getConnection();
            pstmt = con.preparedStatement(sql);
            // substitute the params
            rs = pstmt.executeQuery();

            if (rs.next()) {
                obj = rowMapper.mapRow(rs, 1);
            }

        } catch (SqlException e) {
            throw new DataAccessException(e);
        } finally {
            // close resources
        }

        return obj;
    }
}
````

```

If you want this saved as a downloadable `.md` file, let me know!
```
