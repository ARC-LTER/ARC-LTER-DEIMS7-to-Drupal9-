# ARC-LTER-DEIMS7-to-Drupal9-
A scaled down Drupal 9 web site with a subset of content from Arctic  DEIMS Drupal 7 site.
## Getting Started

It is assumed that a [LAMP](https://tecadmin.net/install-lamp-ubuntu-20-04/) stack has already been installed.

### Installing Drupal 9 with composer

   **Ubuntu** command-line,
 1. **First install composer using this web page**  

    * https://www.digitalocean.com/community/tutorials/how-to-install-and-use-composer-on-ubuntu-20.04 
    * Composer is recommended for installing Drupal as it will manage Drupal and all dependencies (modules, themes, libraries).
		* The Composer executable should ultimately end up in /usr/local/bin with permissions *chmod 0755*.
		* You will **not** want to run Composer with "sudo", so all users should be able to execute it.
		* Install drush launcher.  With the drush launcher you can simply type drush on the command line. [Instructions for drush launcher](https://github.com/drush-ops/drush-launcher) 
	
2. **Install Drupal 9 site in a testing environment** - See [Overview of installing Drupal with composer](https://www.drupal.org/docs/develop/using-composer/using-composer-to-install-drupal-and-manage-dependencies)  
	Summary of steps:  
  
    * Change permissions for the install directory so you can install without "sudo" (e.g. /var/www with *chmod 0777* or change the owner to user installing drupal)
    * Download drupal/recommended-project composer.json and composer.lock from https://github.com/drupal/recommended-project to /var/www
    * Do a modified installation. https://www.drupal.org/docs/develop/using-composer/using-composer-to-install-drupal-and-manage-dependencies#s-to-do-a-modified-install 
		* For latest Drupal 9 version.
		 
		```
		cd /var/www
		composer create-project drupal/recommended-project:^9.3.1 my_site_name_dir 
		```
		* Where 'my_site_name_dir' is the name of your D9 site, e.g.  arclter.
		* Note: composer will create the site 'newsite' directory and several subdirectories and a new 'composer.json' file in the 'newsite' directory. 
		  (drupal root will be in newsite/web)
		* Modify the /var/www/newsite/composer.json with an editor to include the following list of required projects 
		```
		
        "drupal/adaptivetheme": "^4.1",
        "drupal/address": "^1.9",
        "drupal/at_tool": "^1.4",
        "drupal/backup_migrate": "^5.0",
        "drupal/blog": "^3.0",
        "drupal/devel": "^4.1",
        "drupal/ds": "^3.13",
        "drupal/filehash": "^1.7",
        "drupal/geofield": "1.36",
        "drupal/geofield_map": "^2.71",
        "drupal/imce": "^2.4",
        "drupal/key_value_field": "^1.2",
        "drupal/leaflet": "^2.1",
        "drupal/migrate_plus": "^5.2",
        "drupal/migrate_source_csv": "^3.4",
        "drupal/migrate_tools": "^5.1",
        "drupal/migrate_upgrade": "^3.2",
        "drupal/pathauto": "^1.8",
        "drupal/token": "^1.10",
        "drupal/views_bulk_operations": "^4.0"
		```
		* Review the above list to check if newer versions are available. 
		* Run the following to create the site. 
		
		```
		Composer update
		```
		* Drush 10 is now installed in vendor/bin/drush. If the drush launcher is installed you can just type drush in the 'newsite' directory .  
		
  3. Add a virtual host in apache2 for the ‘newsite’ URL. Document root will be ‘/var/www/newsite/web’. Modify the directive for symbolic links. Add web site user to www-data group.  
 4. In your browser go to your new site. You should arrive at a Drupal set-up page to configure the database, etc.
 5. Reset permissions to something more secure. [Here is some guidance from drupal.org](https://www.drupal.org/node/244924)
 6. Enable the telephone, datetime range, migration, key_value modules and other desired modules using drush or the web-interface but don't enable the migration examples, they only clutter up the database
7. On the command line, navigate to your web root folder. Everything is working well if the command `drush migrate:status` returns all migrate commands.
8. In the settings.php (at e.g., newsite/web/sites/default) file add the database connection information to access the DEIMS7 database. Make sure to call it: migration_source_db, which is used throughout this migration. 
For enhanced security specify the 'config_sync_directory' to be outside the web directory.
	```
	$settings['config_sync_directory'] = '../config/sync';
	
	$databases['migration_source_db']['default'] = array (
		'database' => 'database_name',
		'username' => 'user_name',
		'password' => 'user_password',
		'prefix' => '',
		'host' => 'localhost',
		'port' => '3306',
		'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
		'driver' => 'mysql',
	);
	```
 9. For content .yml files you will need to create .yml files using 'drush migrate-upgrade'. (Before this step you should review your D7 site and decide which taxonomy vocabularies, content types, users you will migrate and/or rename.)
	* Create the config/sync directory `mkdir /var/www/newsite/config/sync`
	* Run command `drush migrate-upgrade --legacy-db-key=migration_source_db --legacy-root=http://deims7domainname --configure-only`
	* Export the migrations using `drush config:export` The .yml files will be in /var/www/newsite/config/sync. 
	* You only need to copy and edit the files that begin with migrate_plus.migration and are not in the install folder. 

	
10. Create the custom module 'deims_migrate' by creating a folder under your new D9 newsite/web/modules/custom (e.g., newsie/web/modules/custom/deims_migrate). Copy the file: deims_migrate.info.yml into this folder. Create a 'config/install' folder inside the 'deims_migrate' folder. The install directory is where .yml files are placed. Copy the .yml files from Deims7-8-Migration/YMLmigration_sripts/modules/custom/deims_migrate/config/install/ to your install folder.  You will need to edit them for any differences. See the next step below.
