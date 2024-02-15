#!/bin/bash

print_green() {
  echo -e "\e[32m$1\e[0m"
}
print_red() {
  echo -e "\e[31m$1\e[0m"
}
if [[ $EUID -ne 0 ]]; then
  print_red "This script must be run as root."
  exit 1
fi

print_green "Updating package index and upgrading installed packages..."
apt-get update && apt-get upgrade -y

# Install required packages for PHP, MySQL, and WordPress
print_green "Installing prerequisites..."
apt-get install -y software-properties-common
add-apt-repository -y ppa:ondrej/php # For latest PHP version
apt-get update
apt-get install -y php7.4 php7.4-mysql php7.4-curl php7.4-gd php7.4-mbstring php7.4-xml php7.4-zip php7.4-xmlrpc

# Install MySQL server and set the root password (replace 'your_mysql_password' with your desired password)
print_green "Installing MySQL server..."
apt-get install -y mysql-server
mysql_secure_installation

print_green "Installing Apache web server..."
apt-get install -y apache2
a2enmod rewrite
systemctl restart apache2

# Download and extract WordPress
print_green "Downloading and extracting WordPress..."
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz -C /var/www/html/
chown -R www-data:www-data /var/www/html/wordpress
chmod -R 755 /var/www/html/wordpress

print_green "WordPress installation is complete."
rm /tmp/latest.tar.gz
