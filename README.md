# remoteValidate - Ajax Validation for AngularJS

[Fork From webadvanced/ng-remote-validate](https://github.com/webadvanced/ng-remote-validate)  

В оригинале для нескольких проверок нужно слать несколько запросов, что не тру.  
По мне лучше - в одном запросе передать по каким правилам прогнать.  

Так же переименовал с **ngRemoteValidate** в **remoteValidate** - с **NG** пускай начинаются родные **Angular** модули, дабы избежать путаницы или пересечения имен в будущем.  
С севера возвращать **isValid**&**message** вместо **isValid**&**value** - первое как то логичнее, на мой взгляд.  

##Как использовать##

Практически все тоже самое, что и у оригинала.  
Загружаем:  

```html
<script type="text/javascript" src="../your/path/remoteValidate.js"></script>
```

Подключаем:

```javascript
var app = angular.module( 'myApp', [ 'remoteValidate' ] );
```  
  
  
**HTML**  

```html
<input type="text" name="login" 
ng-model="user.login" 
remote-validate="( '/ajax/validation/login', ['not_empty',['min_length',2],['max_length',32],'domain','unique'] )" 
required
/>
<br/>
<div class="form-input-valid" ng-show="form.login.$pristine || (form.login.$dirty && rv.login.$valid)">
    От 2 до 16 символов (цифры, латинские буквы и дефис)
</div>
<span class="form-input-valid error" ng-show="form.login.$error.remoteValidate">
    <span ng:bind="form.login.$message"></span>
</span>
```

**BackEnd [Kohana]**   

```php
public function action_validation(){

    $field = $this->request->param('field');
    $value = Arr::get($_POST,'value');
    $rules = Arr::get($_POST,'rules',[]);

    $aValid[$field] = $value;
    $validation = Validation::factory($aValid);
    foreach( $rules AS $rule ){
        if( in_array($rule,['unique']) ){
            /// Clients - Users Models
            $validation = $validation->rule($field,$rule,[':field',':value','Clients']);
        }
        elseif( is_array($rule) ){ /// min_length, max_length
            $validation = $validation->rule($field,$rule[0],[':value',$rule[1]]);
        }
        else{
            $validation = $validation->rule($field,$rule);
        }
    }

    /// В try-catch так как может не существовать такого метода валидации
    $c = false;
    try{
        $c = $validation->check();
    }
    catch( Exception $e ){
        $err = $e->getMessage();
        Response::jEcho($err);
    }

    if( $c ){
        $response = [
            'isValid' => TRUE,
            'message' => 'GOOD'
        ];
    }
    else{
        $e = $validation->errors('validation');
        $response = [
            'isValid' => FALSE,
            'message' => $e[$field]
        ];
    }
    Response::jEcho($response);
}
```

