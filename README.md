
Setup
-----

- install Postgres: https://www.postgresql.org/download/
- install PgAdmin: https://www.pgadmin.org/download/
- create user: `create user postgres with superuser login password 'postgres'`
- create database: `sudo -u postgres createdb stackoverflow`

- Stackexchange data dump: https://archive.org/details/stackexchange
    - Download any dump - example: https://ia800107.us.archive.org/27/items/stackexchange/dba.stackexchange.com.7z
- Import the dumped data into postgres db
    - Extract to `~/sqlqueries/data/`
    - get scripts to load data
	    - `git clone https://github.com/Networks-Learning/stackexchange-dump-to-postgres ~/sqlqueries/download-tool`
	    - `cd ~/sqlqueries/download-tool`
	    - create venv: `python3 -m venv venv`
	    - activate: `source venv/bin/activate`
	    - install requirements:
	        ```py
	        python -m pip install --upgrade pip setuptools wheel
	        python -m pip install -r requirements.txt`
	        ```
    - load files
        ```bash
        python load_into_pg.py -f ../data/Badges.xml -t Badges
        python load_into_pg.py -f ../data/Comments.xml -t Comments
        python load_into_pg.py -f ../data/Users.xml -t Users
        # python load_into_pg.py -f ../data/PostLinks.xml -t PostLinks
        python load_into_pg.py -f ../data/Tags.xml -t Tags
        python load_into_pg.py -f ../data/Votes.xml -t Votes
        # python load_into_pg.py -f ../data/PostHistory.xml -t PostHistory
        python load_into_pg.py -f ../data/Posts.xml -t Posts
        ```
- Start PgAdmin and connect to local server
- Connect to `stackoverflow` database
- Drop all user indexes
    ```sql
    CREATE OR REPLACE FUNCTION drop_all_indexes() RETURNS INTEGER AS $$
    DECLARE
      i RECORD;
    BEGIN
      FOR i IN 
        (SELECT relname FROM pg_class
           -- exclude all pkey, exclude system catalog which starts with 'pg_'
          WHERE relkind = 'i' AND relname NOT LIKE '%_pkey%' AND relname NOT LIKE 'pg_%')
      LOOP
        -- RAISE INFO 'DROPING INDEX: %', i.relname;
        EXECUTE 'DROP INDEX ' || i.relname;
      END LOOP;
    RETURN 1;
    END;
    $$ LANGUAGE plpgsql;

    select drop_all_indexes();
    ```
-- --

Practice Questions
-----------------------
1. Add all required foreign key constraints
	```
	posts.owneruserid       => users.id
	posts.lasteditoruserid  => users.id
	
	votes.postid            => posts.id
	votes.userid            => users.id
	
	comments.postid         => posts.id
	comments.userid         => users.id
	
	badges.userid           => users.id
	```
2. find top 5 `users` with most `posts`
3. find the most frequently given `badge`
4. find all `users` who have not made any `posts`
5. if the above query is frequently made, how will you optimize it?
6. get a uniform random sample of 100 `users` from the users table, and store into `chosen_users` table
7. find all `users` with `displayname` starting with `luffy`
8. find all `users` with `displayname` ending in `sanji`
9. how will you optimize the above query?
10. find the `post` which has been `commented` on by most `users`
11. find the `user` with most `badges`
12. find the `post` with most total `votes` 
13. find all questions with no accepted answers.
14. find `user` who has made the most accepted answers
15. overlapp b/w two `users` is defined as `(the number of posts both users have participated in) / (total number of distinct posts both users have participated in)`. Find the top 5 users with most overlap.
