# sistema gestão de usuários

1) objetivo do projeto
Este projeto foi desenvolvido como parte de uma etapa técnica para demonstrar o domínio da arquitetura MVC (Model-View-Controller) utilizando o framework Laravel. A aplicação consiste em um sistema funcional para cadastro e listagem de usuários.

=======================

2) instruções de instalação

*instalação laravel
sudo apt update
sudo apt install composer
composer create-project laravel/laravel gestao_usuarios
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable --now postgresql

*instalação banco
sudo -u postgres psql
CREATE DATABASE laravel;
CREATE USER laravel WITH ENCRYPTED PASSWORD 'x';
GRANT ALL PRIVILEGES ON DATABASE laravel TO laravel;
\c laravel
ALTER SCHEMA public OWNER TO laravel;
\q

*configuração banco
cd gestao_usuarios
sudo nano .env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=x
sudo systemctl reload postgresql@16-main.service

(frontend CSS e JavaScript)
sudo apt install curl -y
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm install
npm run dev (inicia a página padrão do servidor frontend vite)

--------------

(banco dados)

php artisan migrate (cria a tabela no banco)
database/migrations/cadastro-usuarios_create_users_table.php
php artisan make:migration create_users_table
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email')->unique();
        $table->string('password');
        $table->timestamps();
    });
}

app/Models/User.php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
class User extends Authenticatable
{
    use HasFactory;
    // Campos que podem ser preenchidos em massa
    protected $fillable = [
        'name',
        'email',
        'password',
    ];
    // Ocultar campos sensíveis em arrays/JSON
    protected $hidden = [
        'password',
    ];
}

---------------------

(backend)

*controller
php artisan make:controller UserController (cria controller)

app/Http/Controllers/UserController.php
namespace App\Http\Controllers;
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
class UserController extends Controller
{
    // READ: Retorna a lista de usuários em JSON
    public function index()
    {
        $users = User::all();
        
        // O Laravel converte automaticamente coleções do Eloquent para JSON
        return response()->json($users, 200);
    }

    // CREATE: Recebe o JSON do Insomnia e salva no banco
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => 'required|min:6',
        ]);

        $user = User::create([
            'name' => $validated['name'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password']),
        ]);

        return response()->json([
            'message' => 'Usuário cadastrado com sucesso!',
            'user' => $user
        ], 201); // 201 é o código HTTP padrão para "Criado"
    }

    // UPDATE: Recebe o JSON do Insomnia e atualiza o registro
    public function update(Request $request, User $user)
    {
        $validated = $request->validate([
            'name' => 'sometimes|required|string|max:255',
            'email' => 'sometimes|required|email|unique:users,email,' . $user->id,
            'password' => 'nullable|min:6',
        ]);

        // Atualiza os dados básicos
        $user->name = $request->name ?? $user->name;
        $user->email = $request->email ?? $user->email;

        // Atualiza a senha apenas se foi enviada
        if ($request->filled('password')) {
            $user->password = Hash::make($request->password);
        }

        $user->save();

        return response()->json([
            'message' => 'Usuário atualizado com sucesso!',
            'user' => $user
        ], 200);
    }

    // DELETE: Remove o usuário e retorna a confirmação
    public function destroy(User $user)
    {
        $user->delete();

        return response()->json([
            'message' => 'Usuário excluído com sucesso!'
        ], 200);
    }
}

*rotas (conectam as URLs do navegador aos métodos do controller/api)

routes/api.php
use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;
Route::get('/users', [UserController::class, 'index']);
Route::post('/users', [UserController::class, 'store']);
Route::put('/users/{user}', [UserController::class, 'update']);
Route::delete('/users/{user}', [UserController::class, 'destroy']);

(lista as rotas e confirma a URL)
php artisan route:list

-------------------

(frontend)

resources/views/users/index.blade.php (lista usuários)
<h1>Lista de Usuários</h1>
<a href="{{ route('users.create') }}">Novo Usuário</a>
@if(session('success'))
    <p style="color: green;">{{ session('success') }}</p>
@endif
<table>
    <thead>
        <tr>
            <th>Nome</th>
            <th>Email</th>
        </tr>
    </thead>
    <tbody>
        @foreach($users as $user)
            <tr>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
            </tr>
        @endforeach
    </tbody>
</table>

resources/views/users/create.blade.php (cadastra usuários)
<h1>Cadastrar Usuário</h1>
<form action="{{ route('users.store') }}" method="POST">
    @csrf <label>Nome:</label>
    <input type="text" name="name" value="{{ old('name') }}">
    @error('name') <span>{{ $message }}</span> @enderror
    <br>
    <label>Email:</label>
    <input type="email" name="email" value="{{ old('email') }}">
    @error('email') <span>{{ $message }}</span> @enderror
    <br>
    <label>Senha:</label>
    <input type="password" name="password">
    @error('password') <span>{{ $message }}</span> @enderror
    <br>
    <button type="submit">Salvar</button>
</form>
<a href="{{ route('users.index') }}">Voltar</a>

<h1>Editar Usuário</h1>
<form action="{{ route('users.update', $user->id) }}" method="POST">
    @csrf
    @method('PUT') <label>Nome:</label>
    <input type="text" name="name" value="{{ old('name', $user->name) }}">
    @error('name') <span style="color: red;">{{ $message }}</span> @enderror
    <br><br>
    <label>Email:</label>
    <input type="email" name="email" value="{{ old('email', $user->email) }}">
    @error('email') <span style="color: red;">{{ $message }}</span> @enderror
    <br><br>
    <label>Nova Senha (deixe em branco para manter a atual):</label>
    <input type="password" name="password">
    @error('password') <span style="color: red;">{{ $message }}</span> @enderror
    <br><br>
    <button type="submit">Atualizar</button>
</form>
<br>
<a href="{{ route('users.index') }}">Voltar para a lista</a>

<table border="1" cellpadding="10">
    <thead>
        <tr>
            <th>Nome</th>
            <th>Email</th>
            <th>Ações</th> </tr>
    </thead>
    <tbody>
        @foreach($users as $user)
            <tr>
                <td>{{ $user->name }}</td>
                <td>{{ $user->email }}</td>
                <td>
                    <a href="{{ route('users.edit', $user->id) }}">Editar</a> 
                    
                    <form action="{{ route('users.destroy', $user->id) }}" method="POST" style="display:inline;">
                        @csrf
                        @method('DELETE') <button type="submit" onclick="return confirm('Tem certeza que deseja excluir?')">Excluir</button>
                    </form>
                </td>
            </tr>
        @endforeach
    </tbody>
</table>

-------------

dependencias:
*instalação php
sudo apt install php-xml php-mbstring php-curl php-zip php-pgsql -y

*instalação pacote starter kits
composer require laravel/breeze --dev

*gera chave da aplicação
php artisan key:generate

*gera estrutura dos arquivos views, controllers e rotas
php artisan breeze:install

Blade with Alpine (stack pura laravel)
Pest (framework)

*banco dados
php artisan migrate (cria a tabela)
php artisan make:model User -m (cria o arquivo app/Models/User.php e database/migrations/xxxx_xx_xx_create_users_table.php)

*controller
php artisan make:controller UserController (cria o controller que gerencia a lógica dos usuários e o arquivo app/Http/Controllers/UserController.php para métodos index, create e store)

*view
(cria as views)
php artisan make:view users.index
php artisan make:view users.create

*inicia o servidor
php artisan serve

*acesso
http://localhost:8000/users

----------

git init
git add .
git commit -m "sistema gestão usuários"
git remote add origin https://github.com/thiagosilvadevops/sistema-gestao-usuarios
git branch -M main
git push -u origin main

==============

3) modo de utilização

*abaixar o projeto
git clone https://github.com/thiagosilvadevops/sistema-gestao-usuarios
cd gestao_usuarios
composer install (dependências do laravel)
php artisan key:generate (gera chave da aplicação)
php artisan migrate (cria tabelas no banco dados)
php artisan serve (inicia a aplicação)

-------------

*testar o funcionamento

(cria usuário)
Request Collection
Método: POST
URL: http://localhost:8000/api/users
Body: {
  "name": "João da Silva",
  "email": "joao@email.com",
  "password": "senha_segura_123"
}

(lista usuário)
Método: GET
URL: http://localhost:8000/api/users

(atualiza usuário)
Método: PUT
URL: http://localhost:8000/api/users/1
Body: {
  "name": "João Silva Atualizado",
  "email": "joao.novo@email.com"
}

(deleta usuário)
Método: DELETE
URL: http://localhost:8000/api/users

----------

(vs code)
*extensões
remote ssh
laravel extension pack

*configuração
open folder gestao_usuarios

===================
