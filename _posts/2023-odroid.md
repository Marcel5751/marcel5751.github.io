
# Odroid N2+

There is an apache2 server running by default. You can edit the landing page that you see when accessing the IP address of your odroid via web browser. I use this to show a "landing page" of all the services I run on the Odroid.

```bash
sudo nano /var/www/html/index.html
sudo service apache2 restart
```

