# Restoring the nagiosadmin Account - Hyper-V

### Purpose

Restore the nagiosadmin account and permissions allowing for the correct function of NagiosXI on the new Hyper-V environment.

### Background

Nagios Database Architecture
* MySQL
  - nagiosql - contains core config
  - nagios - contains historical data.
* Postgres
    - nagiosxi 
        + Contains XI specific settings and
        + User accounts/permissions.

After restoring the nagios data on Hyper-V then attempting to login as nagiosadmin, logging uncovered corruption of the nagiosadmin account. Further investigation revealed the user was missing from the Postgres:nagiosxi database. The nagiosadmin account is a critical component for correct function of the monitoring server. Therefore, testing was performed to illustrate the steps required in order to restore the accounts function.

### Proceedure

- [ ] Connect to nagiosxi
    - `ssh nagiosxi`

- [ ] Assume postgres user
    - `su postgres`
    - `cd`

- [ ] List postgres databases
    - `psql -l`

- [ ] Verify list contains:
    - nagiosxi
    - postres

- [ ] Launch the postgres client
    - `psql nagiosxi`

- [ ]  List tables in nagiosxi
    - `\d`

- [ ]  Verify list contains:
    - xi_users
    - xi_usermeta

- [ ]  List postgres roles
    - `select * from pg_roles;`

- [ ]  Verify list contains:
    - postgres
    - nagiosxi

- [ ]  If list contains any other roles, eg.. nagios
    - `drop role nagios;`

- [ ] List nagiosxi.xi_users and verify nagiosadmin account is not listed
   - `select * from xi_users;`

- [ ] Create nagiosadmin user
   - `INSERT INTO xi_users (user_id, username, password, name, email, backend_ticket, enabled)`
     `VALUES ( 18 , 'nagiosadmin', MD5( 'nagiosxi' ) , 'nagios', 'root@localhost', NULL , '1' );`

- [ ] Insert dashboards key:
   - `INSERT INTO xi_usermeta (usermeta_id, user_id, keyname, keyvalue, autoload)`
     `VALUES ( 1279, 18, 'dashboards', 'a:3:{i:0;a:4:{s:2:"id";s:6:"screen";s:5:"title";s:10:"[ Screen ]";s:4:"opts";N;s:8:"dashlets";a:0:{}}i:1;a:4:{s:2:"id";s:4:"home";s:5:"title";s:9:"Home Page";s:4:"opts";a:1:{s:10:"background";s:6:"ffffff";}s:8:"dashlets";a:3:{i:1;a:5:{s:2:"id";s:8:"t70nntm8";s:4:"name";s:19:"xicore_server_stats";s:5:"title";s:12:"Server Stats";s:4:"opts";a:6:{s:3:"top";i:324;s:4:"left";i:4;s:6:"zindex";s:1:"4";s:6:"pinned";i:1;s:6:"height";i:382;s:5:"width";i:259;}s:4:"args";a:0:{}}i:6;a:5:{s:2:"id";s:8:"euohqtr8";s:4:"name";s:18:"xicore_admin_tasks";s:5:"title";s:20:"Administrative Tasks";s:4:"opts";a:6:{s:3:"top";i:43;s:4:"left";i:2;s:6:"zindex";s:1:"2";s:6:"height";i:244;s:5:"width";i:330;s:6:"pinned";i:1;}s:4:"args";a:0:{}}i:7;a:5:{s:2:"id";s:8:"n745sfa0";s:4:"name";s:22:"xicore_getting_started";s:5:"title";s:21:"Getting Started Guide";s:4:"opts";a:6:{s:3:"top";i:42;s:4:"left";i:351;s:6:"zindex";s:1:"3";s:6:"height";i:456;s:5:"width";i:337;s:6:"pinned";i:1;}s:4:"args";a:0:{}}}}i:2;a:4:{s:2:"id";s:8:"d7mccig7";s:5:"title";s:15:"Empty Dashboard";s:4:"opts";a:1:{s:10:"background";s:6:"ffffff";}s:8:"dashlets";a:0:{}}}', 0  );` 

- [ ] Insert userlevel key:
   - `INSERT INTO xi_usermeta (usermeta_id, user_id, keyname, keyvalue, autoload)`
         `VALUES ( 1287, 18, 'userlevel', 255, 1 );`

- [ ] Insert authorized_for_all_objects key:
   - `INSERT INTO xi_usermeta (usermeta_id, user_id, keyname, keyvalue, autoload)`
         `VALUES ( 1287, 18, 'authorized_for_all_objects', 1, 1 );`

- [ ] Verify nagiosadmin account's security settings
   - `select usermeta_id, keyname, keyvalue from xi_usermeta where user_id = 18 order by usermeta_id ;`

| KeyName                     | KeyValue                                                    |
| --------------------------- | ----------------------------------------------------------- |
| userlevel                   | 255                                                         |
| dasboards                   | *This will produce a long list, make sure admin is present* |
| authorized_for_all_objects  | 1                                                           |

- [ ] Login to web interface then confirm Authorization Level: Admin
   - Select the Admin tab then,
   - Select Manage Users
   - Edit nagiosadmin

