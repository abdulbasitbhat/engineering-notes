# Method Chaining and Builder pattern Example: SQL Query Builder in Java

This shows how method chaining can be used to build an SQL query in a clean, readable way.

---

## SQL Query Builder Using Chaining (Mutable Style)

    class SqlQueryBuilder {
        private String table;
        private String whereClause;
        private String orderBy;
        private Integer limit;

        public SqlQueryBuilder selectFrom(String table) {
            this.table = table;
            return this;
        }

        public SqlQueryBuilder where(String condition) {
            this.whereClause = condition;
            return this;
        }

        public SqlQueryBuilder orderBy(String column) {
            this.orderBy = column;
            return this;
        }

        public SqlQueryBuilder limit(int limit) {
            this.limit = limit;
            return this;
        }

        public String build() {
            StringBuilder sql = new StringBuilder();
            sql.append("SELECT * FROM ").append(table);

            if (whereClause != null) {
                sql.append(" WHERE ").append(whereClause);
            }
            if (orderBy != null) {
                sql.append(" ORDER BY ").append(orderBy);
            }
            if (limit != null) {
                sql.append(" LIMIT ").append(limit);
            }

            return sql.toString();
        }
    }

---

## Using the Builder

    String sql =
            new SqlQueryBuilder()
                    .selectFrom("users")
                    .where("age > 18")
                    .orderBy("name")
                    .limit(10)
                    .build();

    // SELECT * FROM users WHERE age > 18 ORDER BY name LIMIT 10

---

## Why Chaining Works Here

- Each method returns `this`
- Internal state is updated step by step
- Final `build()` produces the result

---
