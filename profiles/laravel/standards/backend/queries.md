## Database query best practices

- **Prevent SQL Injection**: Always use parameterized queries or Eloquent; never interpolate user input into SQL strings
- **Avoid N+1 Queries**: Use eager loading to avoid N+1 queries when possible
- **Select Only Needed Data**: Request only the columns you need rather than using SELECT * for better performance
- **Index Strategic Columns**: Index columns used in WHERE, JOIN, and ORDER BY clauses for query optimization
- **Use Transactions for Related Changes**: Wrap related database operations in transactions to maintain data consistency
- **Cache Expensive Queries**: Cache results of complex or frequently-run queries when appropriate
