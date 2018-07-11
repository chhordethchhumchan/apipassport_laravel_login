# apipassport_laravel_login
#####  For Login 
'headers' => [

    'Accept' => 'application/json',

    'Authorization' => 'Bearer '.$accessToken,

]

Step 1 : Install Laravel
  composer create-project --prefer-dist laravel/laravel blog
Step 2: Install Package
  composer require laravel/passport
#####
After successfully install package, open config/app.php file and add service provider.

config/app.php

'providers' => [

	....

	Laravel\Passport\PassportServiceProvider::class,

],

### Step 3: Run Migration and Install
  php artisan migrate
  Next, we need to install passport using command, Using passport:install command, it will create token keys for security. So let's run bellow command:
  php artisan passport:install
  
 ####Step 4: Passport Configuration
 * app/User.php
    + use Laravel\Passport\HasApiTokens;
    + use HasApiTokens, Notifiable;
 * app/Providers/AuthServiceProvider.php
    use Laravel\Passport\Passport;
    Input in boot function 
     public function boot()
    {
    ....
        Passport::routes();
    ....
 *#config/auth.php

<?php


return [
    .....
    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],
    .....
]
    }
    
  ####Step 5: Create API Route
Route::post('login', 'API\UserController@login');
Route::post('register', 'API\UserController@register');


Route::group(['middleware' => 'auth:api'], function(){
	Route::post('details', 'API\UserController@details');
});
###Step 6: Create Controller
app/Http/Controllers/API/UserController.php

<?php


namespace App\Http\Controllers\API;


use Illuminate\Http\Request;
use App\Http\Controllers\Controller;
use App\User;
use Illuminate\Support\Facades\Auth;
use Validator;


class UserController extends Controller
{


    public $successStatus = 200;


    /**
     * login api
     *
     * @return \Illuminate\Http\Response
     */
    public function login(){
        if(Auth::attempt(['email' => request('email'), 'password' => request('password')])){
            $user = Auth::user();
            $success['token'] =  $user->createToken('MyApp')->accessToken;
            return response()->json(['success' => $success], $this->successStatus);
        }
        else{
            return response()->json(['error'=>'Unauthorised'], 401);
        }
    }


    /**
     * Register api
     *
     * @return \Illuminate\Http\Response
     */
    public function register(Request $request)
    {
        $validator = Validator::make($request->all(), [
            'name' => 'required',
            'email' => 'required|email',
            'password' => 'required',
            'c_password' => 'required|same:password',
        ]);


        if ($validator->fails()) {
            return response()->json(['error'=>$validator->errors()], 401);            
        }


        $input = $request->all();
        $input['password'] = bcrypt($input['password']);
        $user = User::create($input);
        $success['token'] =  $user->createToken('MyApp')->accessToken;
        $success['name'] =  $user->name;


        return response()->json(['success'=>$success], $this->successStatus);
    }


    /**
     * details api
     *
     * @return \Illuminate\Http\Response
     */
    public function details()
    {
        $user = Auth::user();
        return response()->json(['success' => $user], $this->successStatus);
    }
}

##### reference
https://itsolutionstuff.com/post/laravel-5-how-to-create-api-authentication-using-passport-example.html
