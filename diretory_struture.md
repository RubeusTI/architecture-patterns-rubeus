# Estrutura de diretorios:


```
    |-- Interface
    |  |-- Http
    |  |  |-- Controller
    |  |  |  |-- User
    |  |  |  |  |-- Login
    |  |  |-- Kernel
    |  |-- Console.php
    |-- Application
    |-- Domain
    |-- Infrastructure
```

## Interface:

Responsável por receber e tratar todas as entradas do sistema.

<details>
<pre><code> 
namespace App\Application\Http\Controllers;

use Illuminate\Http\Request;
use Validator;
use App\Domain\User\UserAggregation;

class LoginController extends Controller
{

    private $userAggregation;
    
    public function __construct(UserAggregation $userAggregation)
    {
        $this->userAggregation = $userAggregation;
    }

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
        $this->validate($request, $this->roles(), $this->messages());

        $userLogged = $this->userAggregation->login($request->get('login'), $request->get('password'));

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
</code></pre>
</details>


## Application:

Responsável por tratar regras de aplicação. Ex: tratar sequência de ações; dados a serem vinculados a sessão etc..

<details>

</details>

## Domain:

Responsável por tratar regras referentes ao negocio e guarda as entidades relevantes para o negocio

## Infrastruture:

Resposável por persistir e recuperar os dados da aplicação e interface de comunicação com outros recursos extermos a aplicação


[Controller Laravel](https://matthewdaly.co.uk/blog/2018/02/18/put-your-laravel-controllers-on-a-diet/)
[Soft delete](https://laravel.com/docs/5.7/eloquent#soft-deleting)