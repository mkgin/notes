# zabbix


## Admin Account Password Reset

* Via the SQL database.
  * `update zabbix.users set passwd=('$2a$10$hP35PoXlzKbWxlRRhaLYM.wgFvPFY4RLIpWpOKrrM47Q.jOVy3Kqu') where username='Admin';`
    * User: "Admin"
    * Pass: "ChangeMe"
  * Via [r/zabbix](https://www.reddit.com/r/zabbix/comments/qh3apy/comment/jlkw0do/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button) on Reddit.


## LDAP authentication
- LDAP authentication with "Admin" using internal.
  - Authentication set to LDAP.
  - put Admin into a group with "Frontend access: Internal"
  - put Ldap users into a group with "Frontend access: Default"
- https://www.zabbix.com/forum/zabbix-help/36389-zabbix-ldap-and-local-authentication-question
