# Opção 1:

## Interface

definida por xml que serve como base para a documentação e para o o atual "/teste"

algumas observações:
 o transation: é para permitir o controle de traqnsição por etapa

```xml
<?xml version="1.0" encoding="UTF-8"?>

<process method="post" url="login" permission="1" validatePermission="validarPermissao" roleCache="no">
    
    <input>
        <field role="require{{Login  é obrigatorio}}" key="User::login" name="login"/>
        <field role="require{{Password  é obrigatorio}}" key="User::password" name="password"/>
    </input>
    
    <activities>

        <activity key="login" next="create-session" transaction="independent">
            <class name="User\Login" method="login"/>
        </activity>

        <activity key="create-session"  transaction="none">
            <class name="User\Session" method="create"/>
        </activity>

    </activities>
    
    <output>
        <class name="User\Login" method="response"/>
    </output>
    
</process>
```

## Application

A divisão do em duas: uma para chamar o verifiar login e outra para setar os dados na sessão é só para eu ter duas etapas no xml
E o response do Login é só para ter um exemplo de output

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
            $msg->put('User', $user);
            return true;
        }
        return false;
    }

    public function response($msg, $erro=false, $exception=false){
        if (!$exception && !$erro) {
            response()->json($msg->get('response'));
        }
        response()->json($erro ? $erro : $exception);
    }
}

```


```php
<?php

namespace App\Application\User;

use App\Application\Application;

class Session extends Application
{
    
    public function login($msg)
    {        
        $user = $msg->get('User');
        $this->app['session']->put('userId', $user->id);
        $this->app['session']->put('personId', $user->personId);
        $this->app['session']->put('name', $user->name);
        $this->app['session']->save();
        return $user;
    }
}

```


# Domain


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

# Infrastruture

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