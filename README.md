# symfony_doctrine_eg (for MySQL and MongoDB)
# https://symfony.com/doc/current/doctrine.html#main
# https://symfony.com/doc/current/bundles/SensioFrameworkExtraBundle/annotations/converters.html
# http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html

A Symfony project created on December 13, 2016, 8:50 pm.

INSTALLATION CHECKLIST
#1. Install latest Symfony Installer with the following:
source: http://symfony.com/download

    sudo curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony
    sudo chmod a+x /usr/local/bin/symfony

#2. Create new project:
source: http://symfony.com/download

    symfony new my_project

#3. Change permissions of the /var/cache/ directories:
source: http://symfony.com/doc/current/setup/file_permissions.html

    rm -rf var/cache/*
    rm -rf var/logs/*

    HTTPDUSER=`ps axo user,comm | grep -E '[a]pache|[h]ttpd|[_]www|[w]ww-data|[n]ginx' | grep -v root | head -1 | cut -d\  -f1`
    sudo chmod -R +a "$HTTPDUSER allow delete,write,append,file_inherit,directory_inherit" var
    sudo chmod -R +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" var

#4. Setup Doctrine and then the database.
source: http://symfony.com/doc/3.1/doctrine.html

    Within the parameters YAML file add the following parameters:
    # app/config/parameters.yml

    parameters:
        database_host:      localhost
        database_port:      null
        database_name:      doctrine_eg_db2
        database_user:      doctrine_eg_user
        database_password:  doctrine_eg_user
        mailer_transport: smtp
        mailer_host: 127.0.0.1
        mailer_user: null
        mailer_password: null
        secret: (40 digit alphanumeric secret goes here.)


    NOTE #1: In the Symfony documentation it only lists four out of the necessary parameters needed for the database.
             You should use all ten (10) parameters in order for the php command line to work.
    NOTE #2: The parameter information will be referenced in app/config/config.yml

    # app/config/config.yml
    doctrine:
        dbal:
            driver:   pdo_mysql
            host:     '%database_host%'
            dbname:   '%database_name%'
            user:     '%database_user%'
            password: '%database_password%'


#5. Make certain that your mysql configuration is setup to create databases with a default of utf8, not latin type collations.
source: http://symfony.com/doc/3.1/doctrine.html

    On Mac, there might not be a my.cnf file. So go to: /usr/local/mysql/support-files/ in order to checkout the examples.

    vim /usr/local/mysql/support-files/my-default.cnf
    sudo cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
    sudo vim /etc/my.cnf
    (Under [mysqld] add the following three lines):


             # For advice on how to change settings please see
             # http://dev.mysql.com/doc/refman/5.6/en/server-configuration-defaults.html
             # *** DO NOT EDIT THIS FILE. It's a template which will be copied to the
             # *** default location during install, and will be replaced if you
             # *** upgrade to a newer version of MySQL.

             [mysqld]
             # Version 5.5.3 introduced "utf8mb4", which is recommended
             collation-server     = utf8mb4_general_ci # Replaces utf8_general_ci
             character-set-server = utf8mb4            # Replaces utf8

             # Remove leading # and set to the amount of RAM for the most important data
             # cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
             # innodb_buffer_pool_size = 128M

             # Remove leading # to turn on a very important data integrity option: logging
             # changes to the binary log between backups.
             # log_bin

             # These are commonly set, remove the # and set as required.
             # basedir = .....
             # datadir = .....
             # port = .....
             # server_id = .....
             # socket = .....

             # Remove leading # to set options mainly useful for reporting servers.
             # The server defaults are faster for transactions and fast SELECTs.
             # Adjust sizes as needed, experiment to find the optimal values.
             # join_buffer_size = 128M
             # sort_buffer_size = 2M
             # read_rnd_buffer_size = 2M

             sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

#5A. In System Preferences if you have MySQL Server Status installed you can simply restart the MySQL server.
source: http://symfony.com/doc/3.1/doctrine.html

#5B. Update the app/config/config.yml so doctrine is using the correct charset.
source: http://symfony.com/doc/3.1/doctrine.html

    NOTE:   The people at Symfony recommend against MySQL's utf8 character set, since it does not support 4-byte unicode
            characters, and strings containing them will be truncated. This is fixed by the newer utf8mb4 character set.


        #app/config/config.yml

        # Doctrine Configuration
        doctrine:
            dbal:
                driver:   pdo_mysql
                host:     "%database_host%"
                port:     "%database_port%"
                dbname:   "%database_name%"
                user:     "%database_user%"
                password: "%database_password%"
                #charset:  UTF8
                charset: utf8mb4
                default_table_options:
                    charset: utf8mb4
                    collate: utf8mb4_unicode_ci


#6. Create the new databse.
source: http://symfony.com/doc/3.1/doctrine.html

        After setting up the database information in doctrine we can use the following command line to generate the database.

        php bin/console doctrine:database:create

        COMMAND LINE EXAMPLE:

            $ php bin/console doctrine:database:create
            Created database `doctrine_eg_db` for connection named default


#6A. To drop and refresh the database do the following:
source: http://symfony.com/doc/3.1/doctrine.html

        php bin/console doctrine:database:drop --force
        php bin/console doctrine:database:create

        COMMAND LINE EXAMPLE:

            $ php bin/console doctrine:database:drop --force
            Dropped database for connection named `doctrine_eg_db`

            $ php bin/console doctrine:database:create
            Created database `doctrine_eg_db` for connection named default


#7. Creating Entity Classes.
source: http://symfony.com/doc/3.1/doctrine.html

7A.) Create a directory in your src/{bundle name}/ directory.
source: http://symfony.com/doc/3.1/doctrine.html

    It should be on the same level as your Controllers
    directory. Inside this directory you will be creating the entities that will represent the tables in your database.
    These entities are data classes that are created to represent tables within the database and hold the information
    (as well as getters and setters) in order to provide the db information as needed.

        EXAMPLE:
        /src/AppBundle/Entity

7B.) Create your entities like the one in /src/AppBundle/Entity/User.php
source: http://symfony.com/doc/3.1/doctrine.html

        NOTE: The file should begin with capitalization. So in this case, we save the file as User.php, NOT user.php.

        EXAMPLE:
        <?php

        // src/AppBundle/Entity/User.php
        namespace AppBundle\Entity;

        class User
        {
            private $firstName;
            private $lastName;
            private $emalAddress;
            private $userPassword;
            private $dateCreated;
            private $dataModified;
            private $status;

        }

        /* EOF */


7C.) Use interactive mode to create entities:
source: http://symfony.com/doc/3.1/doctrine.html

        NOTE: When using it three (3) files will be created:
            1.) The User.php data class file.
            2.) The User.orm.php file.
            3.) The UserRepository.php file.


        COMMAND LINE:
            php bin/console doctrine:generate:entity


        EXAMPLE 7Ci OUTPUT:

            $ php bin/console doctrine:generate:entity

            Welcome to the Doctrine2 entity generator
            This command helps you generate Doctrine2 entities.

            First, you need to give the entity name you want to generate.
            The Entity shortcut name: AppBundle:Account

            Determine the format to use for the mapping information.
            Configuration format (yml, xml, php, or annotation) [annotation]: php

            Instead of starting with a blank entity, you can add some fields now.
            Note that the primary key will be added automatically (named id).

            Available types: array, simple_array, json_array, object,
            boolean, integer, smallint, bigint, string, text, datetime, datetimetz,
            date, time, decimal, float, binary, blob, guid.

            New field name (press <return> to stop adding fields):

              created ./src/AppBundle/Entity/Account.php
              created ./src/AppBundle/Resources/config/doctrine/Account.orm.php
            > Generating entity class src/AppBundle/Entity/Account.php: OK!
            > Generating repository class src/AppBundle/Repository/AccountRepository.php: OK!
            > Generating mapping file src/AppBundle/Resources/config/doctrine/Account.orm.php: OK!

        EXAMPLE 7Cii OUTPUT: Account.php

                <?php

                namespace AppBundle\Entity;

                use Doctrine\ORM\Mapping as ORM;

                /**
                 * Account
                 */
                class Account
                {
                    /**
                     * @var int
                     */
                    private $id;

                    /**
                     * @var string
                     */
                    private $firstName;

                    /**
                     * @var string
                     */
                    private $lastName;

                    /**
                     * @var string
                     */
                    private $emailAddress;

                    /**
                     * @var string
                     */
                    private $accountPassword;

                    /**
                     * @var \DateTime
                     */
                    private $createDate;

                    /**
                     * @var \DateTime
                     */
                    private $modifiedDate;

                    /**
                     * @var int
                     */
                    private $accountStatus;


                    /**
                     * Get id
                     *
                     * @return int
                     */
                    public function getId()
                    {
                        return $this->id;
                    }

                    /**
                     * Set firstName
                     *
                     * @param string $firstName
                     *
                     * @return Account
                     */
                    public function setFirstName($firstName)
                    {
                        $this->firstName = $firstName;

                        return $this;
                    }

                    /**
                     * Get firstName
                     *
                     * @return string
                     */
                    public function getFirstName()
                    {
                        return $this->firstName;
                    }

                    /**
                     * Set lastName
                     *
                     * @param string $lastName
                     *
                     * @return Account
                     */
                    public function setLastName($lastName)
                    {
                        $this->lastName = $lastName;

                        return $this;
                    }

                    /**
                     * Get lastName
                     *
                     * @return string
                     */
                    public function getLastName()
                    {
                        return $this->lastName;
                    }

                    /**
                     * Set emailAddress
                     *
                     * @param string $emailAddress
                     *
                     * @return Account
                     */
                    public function setEmailAddress($emailAddress)
                    {
                        $this->emailAddress = $emailAddress;

                        return $this;
                    }

                    /**
                     * Get emailAddress
                     *
                     * @return string
                     */
                    public function getEmailAddress()
                    {
                        return $this->emailAddress;
                    }

                    /**
                     * Set accountPassword
                     *
                     * @param string $accountPassword
                     *
                     * @return Account
                     */
                    public function setAccountPassword($accountPassword)
                    {
                        $this->accountPassword = $accountPassword;

                        return $this;
                    }

                    /**
                     * Get accountPassword
                     *
                     * @return string
                     */
                    public function getAccountPassword()
                    {
                        return $this->accountPassword;
                    }

                    /**
                     * Set createDate
                     *
                     * @param \DateTime $createDate
                     *
                     * @return Account
                     */
                    public function setCreateDate($createDate)
                    {
                        $this->createDate = $createDate;

                        return $this;
                    }

                    /**
                     * Get createDate
                     *
                     * @return \DateTime
                     */
                    public function getCreateDate()
                    {
                        return $this->createDate;
                    }

                    /**
                     * Set modifiedDate
                     *
                     * @param \DateTime $modifiedDate
                     *
                     * @return Account
                     */
                    public function setModifiedDate($modifiedDate)
                    {
                        $this->modifiedDate = $modifiedDate;

                        return $this;
                    }

                    /**
                     * Get modifiedDate
                     *
                     * @return \DateTime
                     */
                    public function getModifiedDate()
                    {
                        return $this->modifiedDate;
                    }

                    /**
                     * Set accountStatus
                     *
                     * @param integer $accountStatus
                     *
                     * @return Account
                     */
                    public function setAccountStatus($accountStatus)
                    {
                        $this->accountStatus = $accountStatus;

                        return $this;
                    }

                    /**
                     * Get accountStatus
                     *
                     * @return int
                     */
                    public function getAccountStatus()
                    {
                        return $this->accountStatus;
                    }
                }

        EXAMPLE 7Ciii OUTPUT: Account.orm.php

            <?php

            use Doctrine\ORM\Mapping\ClassMetadataInfo;

            $metadata->setInheritanceType(ClassMetadataInfo::INHERITANCE_TYPE_NONE);
            $metadata->customRepositoryClassName = 'AppBundle\Repository\AccountRepository';
            $metadata->setChangeTrackingPolicy(ClassMetadataInfo::CHANGETRACKING_DEFERRED_IMPLICIT);
            $metadata->mapField(array(
               'fieldName' => 'id',
               'type' => 'integer',
               'id' => true,
               'columnName' => 'id',
              ));
            $metadata->mapField(array(
               'columnName' => 'firstName',
               'fieldName' => 'firstName',
               'type' => 'string',
               'length' => '100',
              ));
            $metadata->mapField(array(
               'columnName' => 'lastName',
               'fieldName' => 'lastName',
               'type' => 'string',
               'length' => '100',
              ));
            $metadata->mapField(array(
               'columnName' => 'emailAddress',
               'fieldName' => 'emailAddress',
               'type' => 'string',
               'length' => '100',
              ));
            $metadata->mapField(array(
               'columnName' => 'accountPassword',
               'fieldName' => 'accountPassword',
               'type' => 'string',
               'length' => '100',
              ));
            $metadata->mapField(array(
               'columnName' => 'createDate',
               'fieldName' => 'createDate',
               'type' => 'datetime',
              ));
            $metadata->mapField(array(
               'columnName' => 'modifiedDate',
               'fieldName' => 'modifiedDate',
               'type' => 'datetime',
              ));
            $metadata->mapField(array(
               'columnName' => 'accountStatus',
               'fieldName' => 'accountStatus',
               'type' => 'integer',
              ));
            $metadata->setIdGeneratorType(ClassMetadataInfo::GENERATOR_TYPE_AUTO);

        EXAMPLE 7Civ OUTPUT: UserRepository.php

            <?php

            namespace AppBundle\Repository;

            /**
             * AccountRepository
             *
             * This class was generated by the Doctrine ORM. Add your own custom
             * repository methods below.
             */
            class AccountRepository extends \Doctrine\ORM\EntityRepository
            {
            }



8.) Make certain that each Entity class is using Doctrine\ORM\Mapping as ORM.
source: http://symfony.com/doc/3.1/doctrine.html

    8A.) EXAMPLE:

                <?php // src/AppBundle/Entity/Account.php

                namespace AppBundle\Entity;

                use Doctrine\ORM\Mapping as ORM;

                /**
                 * @ORM\Entity
                 * @ORM\Table(name="account")
                 */
                class Account



    8B.) Modify the Entity class you created in order to reference the database table and its columns.
        This is done via metadata within the DocBlock annotations:

            EXAMPLE 8Bi.) In the /src/AppBundle/Entity/Account.php file, switch out default reference to Account (LINE 5-7) with:

                    /**
                     * @ORM\Entity
                     * @ORM\Table(name="account")
                     */

                NOTE:   This change is optional.Doctrine will use the class name as the database table if this change is not
                        included.

            EXAMPLE 8Bii.) In the same file name corrections per property to eference the correct column name and attributes.
            Replace:
                /**
                 * @var int
                 */

            With:
                /**
                 * @ORM\Column(type="integer")
                 * @ORM\Id
                 * @ORM\GeneratedValue(strategy="AUTO")
                 */

            NOTE: Reference Doctrine's guidelines for this to edit all of the different columns.
                    http://symfony.com/doc/3.1/doctrine.html#doctrine-field-types



#9.) Optionally, generate the entities of a bundle, or for a particular namespace:
source: http://symfony.com/doc/3.1/doctrine.html

         EXAMPLE 9A.) Generate all entities in the AppBundle
         php bin/console doctrine:generate:entities AppBundle

         EXAMPLE 9B.) Generate all entities of bundles in the Acme namespace.
         php bin/console doctrine:generate:entities Acme


#10.) Create the database tables/schema
source: http://symfony.com/doc/3.1/doctrine.html

    10A.) IMPORTANT! Before running the command line tool first make certain that the bundle is being referenced in the config.yml file.
          source: http://stackoverflow.com/questions/22267998/symfony2-no-metadata-classes-to-process

          The issue is that unless we specify the mapping for the bundle, Doctrine will not be able to find the meta data.

          CHANGE FROM:

                  doctrine:
                    ...
                      orm:
                          auto_generate_proxy_classes: "%kernel.debug%"
                          naming_strategy: doctrine.orm.naming_strategy.underscore
                          auto_mapping: true

              TO:

                  doctrine:
                        ...
                      orm:
                          auto_generate_proxy_classes: "%kernel.debug%"
                          naming_strategy: doctrine.orm.naming_strategy.underscore
                          auto_mapping: true
                          mappings:
                            AppBundle:
                              type: annotation
                              is_bundle: false
                              dir: %kernel.root_dir%/../src/AppBundle/Entity/
                              prefix: AppBundle\Entity
                              alias: AppBundle


10B.) Afterwards we can run the following command.
source: http://symfony.com/doc/3.1/doctrine.html

    COMMAND LINE EXAMPLE:

        php bin/console doctrine:schema:update --force

    EXAMPLE OUTPUT:

        $ php bin/console doctrine:schema:update --force
        Updating database schema...
        Database schema updated successfully! "1" query was executed



WHEN IN DOUBT:
#1.) If you get errors like:
     No route found for "GET /"


        COMMAND LINE:
        1.) Clear the cache.

            php bin/console cache:clear

        2.) Make certain the route is set.

            php bin/console router:debug
        php bin/console server:run


#2.) RE: ANNOTATION
        The AppBundle:Product string is a shortcut you
        can use anywhere in Doctrine instead of the full
        class name of the entity (i.e. AppBundle\Entity\Product).
        As long as your entity lives under the Entity namespace
        of your bundle, this will work.
