- as intruction, run docker compose command with no issue and all containers are up and running.
- only nginx container expose 8080:80 port 
- try to access / path, behave as expected.
- try to access /api/<random path> pass to http://api:3001 , return error. 
- suspect it could be mismatch from nginx config and api port since all containers are running

- deep dive into api, it run on 3000 but nginx pass to 3001 -> solution: correct the port in ./nginx/conf.d/default.conf

- in index.js also have /status path but it cannot be accessed since no matching config in ./nginx/conf.d/default.conf
    - option 1: change /status to /api/status in index.js so that we can reuse existed location block in nginx
    - option 2: adding new location block in ./nginx/conf.d/default.conf that pass /status to api:3000/status