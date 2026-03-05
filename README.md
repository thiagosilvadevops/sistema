# sistema

------------------

#processo instalação/configuração arquitetura MVC

(backend Laravel)
sudo apt update
sudo apt install composer
composer create-project laravel/laravel sistema-cadastro
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable --now postgresql

sudo -u postgres psql
CREATE DATABASE laravel;
CREATE USER laravel WITH ENCRYPTED PASSWORD 'x';
GRANT ALL PRIVILEGES ON DATABASE laravel TO laravel;
\c laravel
ALTER SCHEMA public OWNER TO laravel;
\q

cd sistema-cadastro
sudo nano .env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=x
sudo systemctl reload postgresql@16-main.service

sudo apt install php-xml php-mbstring php-curl php-zip php-pgsql -y
composer require laravel/breeze --dev (instalação pacote starter kits)

php artisan key:generate
php artisan breeze:install (arquivos do sistema login e cadastro - views, controllers e rotas)
Blade with Alpine (stack pura laravel)
Pest (framework)

php artisan migrate (cria a tabela dos usuários no banco)

(frontend CSS e JavaScript)
sudo apt install curl -y
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm install
npm run dev (inicia a página padrão do servidor frontend vite)

(vs code)
*extensões
remote ssh
laravel extension pack

*configuração
open folder sistema-cadastro
