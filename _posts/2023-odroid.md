
# Odroid N2+

I was looking for a reasonably priced alternative to the sold out Raspbarry Pi 4 at the time. After some research, I stumbled upon the Odroid N2+, which seemed powerful enough for my purposes, offering good value for money.

There is an apache2 server running by default. You can edit the landing page that you see when accessing the IP address of your odroid via web browser. I use this to show a "landing page" of all the services I run on the Odroid.

```bash
sudo nano /var/www/html/index.html
sudo service apache2 restart
```

