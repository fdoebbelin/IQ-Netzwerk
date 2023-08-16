## Forgotten admin password

If you have access to the DB you can modify and run this query:
```sql
UPDATE users SET user_hash = '$2y$10$ua6PicOvqyYMKgOR6gzFcub.Z5s40j6moWRH4oaO.Ef667lz.nb0m'
WHERE user_name = 'user';
```

After that try to login with password:  

```
Password123
```
