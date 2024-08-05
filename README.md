1. Build Image: `docker build --pull --rm -f "images/odoo/Dockerfile" -t odoodevdocker:latest "images/odoo"`
2. Download from github Odoo 16 src and copy into folder development/odoo16 with name odoo. 
Example project structure into development folder.
```
odoo16
    custom_addos
    odoo
    odoo.conf
```
`You can repeat this structure into development folder with others odoo versions.`

2. Open VScode and `Reopen in Devcontainer`.
3. Check and ajust `launch.json` and `settings.json` for debug
4. You can connect PgAdmin or other client for PostgreSQL in your computer.
```
user: odoo
pwd: odoo
port: 5433
host: localhost
```
Note: We use 5433 port for avoid any port conflict with the host. This port is only for external connectios for example PgAdmin or others in your computer.