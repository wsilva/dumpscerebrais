+++
title = "Integrando Zend e Doctrine"
categorias = ["Tecnologia"]
tags = ["Doctrine","PHP","Zend"]
date = "2012-09-02T18:05:00-03:00"
+++

Saudações.

Para ilustrar essa integração criei um projeto praticamente do
zero e segui instruções dos sites que cito nas referências ao final
deste artigo.

<!--continua-->

## Arquivos:

- Baixamos o Zend framework no link
[http://framework.zend.com/download/latest](http://framework.zend.com/download/latest) 
(Versão estável na época 1.12.0);

<img class="img-responsive img-thumbnail" title="Download Zend" alt="Download Zend" src='/assets/images/zend-downloads.png' />

- Baixamos o Doctrine no link
[http://www.doctrine-project.org/projects/orm.html](http://www.doctrine-project.org/projects/orm.html) 
(Versão estável na época 2.2.2)

<img class="img-responsive img-thumbnail" title="Download Doctrine" alt="Download Doctrine" src='/assets/images/doctrine-download.png' />

## Hands On:

### Pastas e arquivos:

Criamos uma pasta para o projeto que vamos desenvolver:

    mkdir -p /var/www/nginx/exemplo/

(se estiver usando o nginx como eu)
ou

    mkdir -p /var/www/html/exemplo/

(caso esteja usando apache)

Essa pasta pode estar em qualquer lugar do seu sistema, basta que ela
seja acessível pelo seu webserver e  na configuração do virtual host você
aponte para esta pasta.

Extraímos o conteúdo da pasta /ZendFramework-1.12.0/ que está no arquivo
ZendFramework-1.12.0.tar.gz para a pasta do projeto.

<img class="img-responsive img-thumbnail" title="Arquivos Zend" alt="Arquivos Zend" src='/assets/images/arquivos-zend.png' />

Para ganhar tempo na criação do projeto eu acabei baixando os arquivos do
Tutorial Quick Start neste link
[http://framework.zend.com/demos/ZendFrameworkQuickstart.tar.gz](http://framework.zend.com/demos/ZendFrameworkQuickstart.tar.gz) 
e descompactei dentro da pasta do projeto.

Mas atenção, fiz apenas para ganhar tempo, recomento que siga o tutorial
passo a passo. Com ele você cria um projeto e aprende como as coisas
funcionam com o Zend Framework

### Virtual Host:
Para testar criamos o Virtual Host para o projeto:

Exemplo para nginx:

    server
    {
        listen 80;
        server_name exemplo.inet;
        root /var/www/nginx/exemplo/public;
        index index.php index.html index.htm;

        # unless the request is for a valid file (image, js, css, etc.), send to bootstrap
        if (!-e $request_filename)
        {
            rewrite ^/(.*)$ /index.php?/$1 last;
            break;
        }

        error_page 404 /index.php;

        location ~* \.(jpg|jpeg|gif|png|css|js|ico|xml)$
        {
            access_log off;
            log_not_found off;
            expires 360d;
        }

        location ~* \.php$
        {
            server_tokens off;
            client_max_body_size 20m;
            client_body_buffer_size 128k;

            include /etc/nginx/fastcgi_params;
            try_files $uri $uri/ /index.php?$query_string;
            fastcgi_pass  127.0.0.1:9001;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            fastcgi_param SCRIPT_NAME $fastcgi_script_name;
            fastcgi_param APPLICATION_ENV development;
        }

        # deny access to hidden files as apache .htaccess files
        location ~ /\.
        {
            access_log off;
            log_not_found off;
            deny all;
        }
    }

Exemplo para apache:

    <VirtualHost *:80>
        ServerName exemplo.inet
        DocumentRoot /var/www/html/exemplo/public

        SetEnv APPLICATION_ENV "development"

        <Directory /var/www/html/exemplo/public>
            DirectoryIndex index.php
            AllowOverride All
            Order allow,deny
            Allow from all
        <Directory>
    <VirtualHost>

### Finalizando o projeto:

Adicionamos a linha abaixo no arquivo de hosts (/etc/hosts):

    127.0.0.1    exemplo.inet

## Integrando o Doctrine:

### Arquivos:

Do arquivo baixado (DoctrineORM-2.2.2-full.tar.gz no meu caso) vamos extrair apenas as pastas Doctrine/ e bin/

<img class="img-responsive img-thumbnail" title="Arquivos Doctrine" alt="Arquivos Doctrine" src='/assets/images/arquivos-doctrine.png' />

A pasta bin/ deve ser extraída para dentro da raiz do projeto
*/var/www/nginx/exemplo/* ou */var/www/html/exemplo/*
(Atenção: é provavel que já exista uma pasta bin/ do próprio zendframework,
pode mesclar o conteúdo das duas pastas)

Já a pasta Doctrine/ deve ir dentro da pasta library
*/var/www/nginx/exemplo/library/* ou */var/www/html/exemplo/library/* .

### Configuração:

Aqui estamos configurando para usar um banco de dados MySQL mas para
outros DBs a configuração é análoga.

Alteramos o arquivo de configuração
*application/configs/application.ini* e na área de produção *[production]*
adicionamos as seguintes linhas:

    doctrine.conn.host = '127.0.0.1'
    doctrine.conn.user = 'root'
    doctrine.conn.pass = 'senha-do-seu-bd'
    doctrine.conn.driv = 'pdo_mysql'
    doctrine.conn.dbname = 'dbexemplo'
    doctrine.path.models = APPLICATION_PATH "/models"

Atente para as configurações onde *doctrine.conn.user* é o nome de usuário
que acessa seu banco de dados, doctrine.conn.pass é a senha desse usuário
e *doctrine.conn.dbname* é o nome do seu banco de dados que pode estar
até vazio mas deve ser criado manualmente.

Alteramos também o arquivo *application/Bootstrap.php* adicionando o
método abaixo:

    /**
     * Initialize Doctrine
     * @return Doctrine_Manager
     */
    public function _initDoctrine()
    {
        // include and register Doctrine's class loader
        require_once('Doctrine/Common/ClassLoader.php');
        $classLoader = new \Doctrine\Common\ClassLoader(
            'Doctrine',
            APPLICATION_PATH . '/../library/'
        );
        $classLoader->register();

        // create the Doctrine configuration
        $config = new \Doctrine\ORM\Configuration();

        // setting the cache ( to ArrayCache. Take a look at
        // the Doctrine manual for different options ! )
        $cache = new \Doctrine\Common\Cache\ArrayCache;
        $config->setMetadataCacheImpl($cache);
        $config->setQueryCacheImpl($cache);

        // choosing the driver for our database schema
        // we'll use annotations
        $driver = $config->newDefaultAnnotationDriver(
            APPLICATION_PATH . '/models'
        );
        $config->setMetadataDriverImpl($driver);

        // set the proxy dir and set some options
        $config->setProxyDir(APPLICATION_PATH . '/models/Proxies');
        $config->setAutoGenerateProxyClasses(true);
        $config->setProxyNamespace('App\Proxies');

        // now create the entity manager and use the connection
        // settings we defined in our application.ini
        $connectionSettings = $this->getOption('doctrine');
        $conn = array(
            'driver'    => $connectionSettings['conn']['driv'],
            'user'      => $connectionSettings['conn']['user'],
            'password'  => $connectionSettings['conn']['pass'],
            'dbname'    => $connectionSettings['conn']['dbname'],
            'host'      => $connectionSettings['conn']['host']
        );
        $entityManager = \Doctrine\ORM\EntityManager::create($conn, $config);

        // push the entity manager into our registry for later use
        $registry = Zend_Registry::getInstance();
        $registry->entitymanager = $entityManager;

        return $entityManager;
    }

### Script Doctrine:

Alteramos todo o conteúdo do arquivo *bin/doctrine.php* para o abaixo:

    define('APPLICATION_ENV', 'development');

    define('APPLICATION_PATH', realpath(dirname(__FILE__) . '/../application'));

    set_include_path(implode(PATH_SEPARATOR, array(
        realpath(APPLICATION_PATH . '/../library'),
        get_include_path(),
    )));

    // Doctrine and Symfony Classes
    require_once 'Doctrine/Common/ClassLoader.php';
    $classLoader = new \Doctrine\Common\ClassLoader('Doctrine', APPLICATION_PATH . '/../library');
    $classLoader->register();

    $classLoader = new \Doctrine\Common\ClassLoader('Symfony', APPLICATION_PATH . '/../library/Doctrine');
    $classLoader->register();

    $classLoader = new \Doctrine\Common\ClassLoader('Entities', APPLICATION_PATH . '/models');
    $classLoader->setNamespaceSeparator('_');
    $classLoader->register();

    // Zend Components
    require_once 'Zend/Application.php';

    // Create application
    $application = new Zend_Application(
        APPLICATION_ENV,
        APPLICATION_PATH . '/configs/application.ini'
    );

    // bootstrap doctrine
    $application->getBootstrap()->bootstrap('doctrine');
    $em = $application->getBootstrap()->getResource('doctrine');

    // generate the Doctrine HelperSet
    $helperSet = new \Symfony\Component\Console\Helper\HelperSet(array(
        'db' => new \Doctrine\DBAL\Tools\Console\Helper\ConnectionHelper($em->getConnection()),
        'em' => new \Doctrine\ORM\Tools\Console\Helper\EntityManagerHelper($em)
    ));

    \Doctrine\ORM\Tools\Console\ConsoleRunner::run($helperSet);


### Integração concluída!

<img class="img-responsive img-thumbnail" title="Uhu" alt="Uhu" src='/assets/images/uhu.jpg' />

### Mas e agora?

<img class="img-responsive img-thumbnail" title="Dup" alt="Dup" src='/assets/images/doh.jpg' />

## Testando:

Para um teste básico criamos uma entidade Teste (em *application/models/Teste.php*) com o conteúdo abaixo:

    /**
     * @Entity
     * @Table(name=”testando”)
     */
    class Default_Model_Teste
    {
        /**
         * @Id @Column(type=”integer”)
         * @GeneratedValue(strategy=”AUTO”)
         */
        private $id;

        /** @Column(type=”string”) */
        private $nome;

        public function setNome($string)
        {
            $this->nome = $string;
            return true;
        }
    }

Em seguida rodamos o comando:

    php bin/doctrine orm:schema-tool:update –force

Uma mensagem de sucesso aparece e olhando nosso banco de dados vemos que a
tabela testando foi criada com os campos id e nome.

Criamos o controller (em *application/controllers/TesteController.php*) com
o conteudo abaixo

    class TesteController extends Zend_Controller_Action
    {
        public function init()
        {
            $registry = Zend_Registry::getInstance();
            $this->_em = $registry->entitymanager;
        }
        public function indexAction()
        {
            $testeEntity = new Default_Model_Teste;
            $testeEntity->setNome(‘Yabadabadooo’);
            $this->_em->persist($testeEntity);
            $this->_em->flush();
        }
    }

Ao acessar a url *http://exemplo.inet/teste* vemos que um registro
com *id=1* e *nome = Yabadabadooo* foi criado lá no banco na nossa
tabela *testando*.

## Referências:

- [http://p-celta.blogspot.com.br/2012/04/integracao-do-doctrine-2-com-zend.html](http://p-celta.blogspot.com.br/2012/04/integracao-do-doctrine-2-com-zend.html) 
- [http://www.oelerich.org/integrate-doctrine-2-with-zend-framework-1-11-3/](http://www.oelerich.org/integrate-doctrine-2-with-zend-framework-1-11-3/) 
- [http://net.tutsplus.com/tutorials/php/zend-framework-from-scratch-models-and-integrating-doctrine-orm/](http://net.tutsplus.com/tutorials/php/zend-framework-from-scratch-models-and-integrating-doctrine-orm/) 

Há um projeto interessante ainda digamos na fase alpha onde podemos baixar
o Zend Framework já integrado com o Doctrine, de quebra já tem uma
implementação de ACL e uma camada adicional de negócio que evita que
seus controllers fiquem inchados com um monte de regras de negócio
que se repetem.

Segue a url:

[http://123esqueceoresto.blogspot.com.br/2012/08/projeto-exemplo-integrando-o-zend.html](http://123esqueceoresto.blogspot.com.br/2012/08/projeto-exemplo-integrando-o-zend.html) 

Inté a próxima!
