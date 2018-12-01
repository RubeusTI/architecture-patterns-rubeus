# Opção 1:

## Interface

padrão lavarel

```php
$router->get('/login', ['middleware' => ['Session', 'CorsDomain'], 'uses' => 'LoginController@login']);
```

```php

namespace App\Interfaces\Http\Controllers\User;

use Illuminate\Http\Request;
use Validator;
use App\Interfaces\Http\Controllers\Controller;

class LoginController extends Controller
{
    private function roles(){
        return [
            'login' => 'required',
            'password' => 'required'
        ];
    }

    private function messages(){
        return [
            'login' => [
                'required'  => 'Informe seu login.'
            ],'password' => [
                'required'  => 'Informe sua senha.'
            ]
        ];
    }

    public function login(Request $request)
    {
        $this->validator->make($request->all(),$this->roles(), $this->messages());

        $userLogged = $this->app->make('UserLoginApplication')->login($request->get('login'), $request->get('password'));

        if ($userLogged) {            
            return response()->json($this->responseSuccess($userLogged));
        }
        return response()->json($this->responseFail());
    }

    private function responseSuccess($userLogged){
        return [
            'success' => true,
            'user' => [
                'name' => $userLogged->name
            ],
            'token' => App('session')->getId()
        ];
    }

    private function responseFail(){
        return [
            'success' => false,
            'error' => 'Senha ou usuário inválidos.'
        ];
    }
}

```


## Application

```php
<?php

namespace App\Application\User;

use App\Application\Application;

class Login extends Application
{
    
    public function login($msg)
    {        
        $user = $this->app->make('UserAggregation')->validatePassword($msg->get('User::login'), $msg->get('User::password'));
        
        if ($user) {
            $this->app['session']->put('userId', $user->id);
            $this->app['session']->put('personId', $user->personId);
            $this->app['session']->put('name', $user->name);
            $this->app['session']->save();
            return $user;
        }
        return false;
    }
}

```

## Domain


```php
namespace App\Domain\User;

use App\Domain\User\Model\User;
use App\Infrastructure\User\UserRepository;

class UserAggregation
{
    private $userRepository;

    public function __construct(UserRepository $userRepository){
        $this->userRepository = $userRepository;
    }

    public function validatePassword($login, $pass)
    {        
        $user = $this->userRepository->identify($login);
        
        if ($user->checkPasswod($pass)) {
            return $user;
        }
        return false;
    }

}
```

```php
<?php

namespace App\Domain\User\Model;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class UserPerson extends Model
{
    use UserTrait;
    private $pass;
    private $name;

    public function create($coletion){
        $this->name = $coletion[0]->name;
        $this->pass = $coletion[0]->pass;
        return $this;
    }
}
```

## Infrastruture

```php
<?php

namespace App\Infrastructure\User;

use DB;
use App\Domain\User\Model\UserPerson;

class UserRepository
{
    public function identify($login)
    {
        $data = DB::table('user')
            ->select('user.id', 'person.name', 'user.pass', 'person.id as personId')
            ->join('person', 'person.id', '=', 'user.person_id')
            ->whereRaw('? in (person.email, person.cpf)', [$login])
            ->get();

        return ( new UserPerson() )->create($data);
    }

}
```