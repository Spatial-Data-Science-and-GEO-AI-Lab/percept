# Configuring the Apache2 web server

## Basic setup and SSL

We will not cover all the details of setting up the Apache2 web server with SSL because (a) your distribution probably handles most of it, (b) it is something that is not specific to our software, and (c) there is plenty of [good documentation](https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html) from official sources. We will also not cover setting up your domain name because that is far out of scope for this tutorial and is handled by numerous other generic web site tutorials and instructions.

We will assume that you are using [VirtualHost](https://httpd.apache.org/docs/2.4/vhosts/) to organize your Apache configuration; even if you only host a single domain, it is still useful to do this.

The easiest thing to do on typical Apache installs is to put your configuration in a single file named something like `000-default.conf` (you can choose a more descriptive name if you like, especially if you have multiple virtual hosts and separate configuration files for each one). This goes in the configuration directory, which is usually named `conf.d` and can be found within the Apache configuration directory. On Debian/Ubuntu-based systems that is typically `/etc/apache2/conf.d/` and on RedHat-based systems it is `/etc/httpd/conf.d/` but if you install Apache a different way then you may find configuration files elsewhere.

Be sure to reload your Apache configuration after making changes, `systemctl reload apache2` (Debian/Ubuntu) or `systemctl reload httpd` (RedHat). As usual, you will need root privileges to edit configuration files and run reloading commands.

The remainder of this section assumes you have created a suitable configuration file in the correct configuration directory, and that you have a domain name to fill in for the blank `<domain_name>`.

## mod_proxy

We use `mod_proxy` to set up Apache as a proxy server in front of our application servers. Therefore, ensure that this is installed and enabled.

### Debian / Ubuntu

Run `a2query -m proxy`, which should show output like `proxy (enabled by site administrator)`.

If not, run `a2enmod proxy` and then reload your configuration using `systemctl reload apache2`.

### RedHat / CentOS / EL
You should see output from `httpd -M | grep proxy` showing the `proxy_module`.

If not, edit the file `/etc/httpd/conf.modules.d/00-proxy.conf` and uncomment (remove the leading `#`) the following lines:
      LoadModule proxy_module modules/mod_proxy.so
      LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so
      LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
      LoadModule proxy_http_module modules/mod_proxy_http.so

Then reload your configuration with `systemctl reload httpd`.

## Redirect HTTP to HTTPS

Although this should be part of your basic SSL setup, just in case, we will include an example of a configuration statement that redirects any old-fashioned HTTP requests to the SSL-enabled HTTPS equivalent:

    <VirtualHost *:80>
            ServerName		<domain_name>
            Redirect		"/" "https://<domain_name>/"
    </VirtualHost>

## Configure image server

Before continuing with the configuration, please choose where on your filesystem you would like to keep the files that you wish to serve on the web, also known as `<document_root>`. By default on many distributions these days that is `/var/www/html/`, which is fine if that works for you. Either way, change `<document_root>` to your preferred filesystem path. However, bear in mind that you should make sure that the web server and the imagery database that you set up in the [processing](processing.md) section all agree on the location of the files. Roughly speaking, that means:

    <system_path_base> = <document_root> / <web_path_base>

So, for example, if you wish to have the usual `<document_root> = /var/www/html` and your `<web_path_base> = /img` then your `<system_path_base>` had better be `/var/www/html/img`. It is possible to be more flexible with symlinks, so for example, you could simply ensure that `/var/www/html/img` is a symlink to the actual location of the imagery (presuming your Apache configuration allows `Options FollowSymLinks`, which is likely, but you should consult Apache documentation if this becomes an issue).

    <VirtualHost *:443>
            ServerName		<domain_name>
            DocumentRoot        "<document_root>"
            ProxyPass		<web_path_base>/ "!"
            # Further backend and frontend configuration statements will go here, later:
            # ...
            # The remaining configuration statements should have come from your SSL setup;
            # if they are not present then go back to the Basic setup and SSL section.
            SSLEngine		on
            SSLCertificateFile	<path_to_my_certificate>
            SSLCertificateKeyFile	<path_to_my_key>
            SSLCACertificateFile	<path_to_CA_certificate>
            SSLProtocol		+TLSv1.2
            SSLCipherSuite      ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384
    </VirtualHost>

And reload your web server using the appropriate command described above.

### Testing

In the end, you should be able to access imagery in your web browser. To get a sample image URL at this stage, open up one of the `_insert.sql` files that you generated in the processing stage. For example, run `find <sqldir> -name '*_insert.sql' | head` and open one of the results. Inside you will see an `INSERT` statement and the first value after the `VALUES` keyword should be a sample URL. Open your web browser, type your `<domain_name>` and then paste the sample URL after it. If your domain name has been configured correctly and your web server is running then you should see a street view image pop up.

