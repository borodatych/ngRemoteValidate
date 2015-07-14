# ngRemoteValidate - Ajax Validation for AngularJS

[Fork From webadvanced/ng-remote-validate](https://github.com/webadvanced/ng-remote-validate)

##Getting started - Example##

Grab either the minified version or the standard source from the release folder and add it to your project.

```html
<script type="text/javascript" src="../your/path/ngRemoteValidate.js"></script>
```

**Include ngRemoteValidate in you Angular app**

```javascript
var app = angular.module( 'myApp', [ 'remoteValidation' ] );
```  
  
  
**HTML Example**  

```html
<input type="text" name="login" 
ng-model="user.login" 
ng-remote-validate="( '/ajax/validation/login', ['not_empty',['min_length',2],['max_length',32],'domain','unique'] )" 
required
/>
<br/>
<div class="form-input-valid" ng-show="form.login.$pristine || (form.login.$dirty && rv.login.$valid)">
    От 2 до 16 символов (цифры, латинские буквы и дефис)
</div>
<span class="form-input-valid error" ng-show="form.login.$error.ngRemoteValidate">
    <span ng:bind="form.login.$message"></span>
</span>
```

**BackEnd Example [Kohana]**   

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

