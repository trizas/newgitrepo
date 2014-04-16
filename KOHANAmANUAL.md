# kohana Manual #

## 1. Folders##

 **Inheritance**

[http://kohanaframework.org/3.3/guide/kohana/files#cascading-filesystem](http://kohanaframework.org/3.3/guide/kohana/files#cascading-filesystem "cascading filesystem")



----------

## 2. request Flow ##

Every application follows the same flow:

1. Application starts from index.php.

	1.The application, module, and system paths are set. (APPPATH, MODPATH, and SYSPATH)

	2.Error reporting levels are set.

	3.Install file is loaded, if it exists.

	4.The bootstrap file, APPPATH/bootstrap.php, is included.
	

2. Once we are in bootstrap.php:

	1.The Kohana class is loaded.

	2.Kohana::init is called, which sets up error handling, caching, and logging.

	3.Kohana_Config readers and Kohana_Log writers are attached.

	4.Kohana::modules is called to enable additional modules.
		

	- Module paths are added to the cascading filesystem.


	- Includes each module's init.php file, if it exists.

	- The init.php file can perform additional environment setup, including adding routes.
	 
	5.Route::set is called multiple times to define the application routes.

	6.Request::instance is called to start processing the request.



	- Checks each route that has been set until a match is found.


	- Creates the controller instance and passes the request to it.


	- Calls the Controller::before method.


	- Calls the controller action, which generates the request response.


	- Calls the Controller::after method.
	
	The above 5 steps can be repeated multiple times when using HMVC sub-requests.

3. Application flow returns to index.php.

	
	1. The main Request response is displayed


----------



## 3. MODEL, VIEW, CONTROLLER ##

- **controllers** A Controller is a class file that stands in between the models and the views in an application. It passes information on to the model when data needs to be changed and it requests information from the model when data needs to be loaded. Controllers then pass on the information of the model to the views where the final output can be rendered for the users. Controllers essentially control the flow of the application. Controllers are called by the Request::execute() function based on the Route that the url matched.
 
  	 Controllers can extend other controllers. You can have a controller extend another controller to share common things, such as requiring you to be logged in to use all of those controllers.

	
	- actions - You create actions for your controller by defining a public function with an action_ prefix. Any method that is not declared as public and prefixed with action_ can NOT be called via routing. An action method will decide what should be done based on the current request, it controls the application. 
	
	

	- before+ after -


	- Methods
	
	- Magic methods

	- ::parent / before /after

	- _construct -- this method is called automatically whenever a new object is created

	 _CLASS_ --returns the name of the class. It is a magic constant

	_destruct --to explicitly trigger the destructor, you can destroy the object using function unset


- **models** folder hosts our custom models class. here you create objects and query the database

	Objects are an instance of a class


	Create an instance of the model in your controller to make it accessible. This can be done directly in a controller method as $user = new User_Model. If you want the model accessible from all of your controller methods, create an instance of the model in your controller constructor: $this→user = new User_Model.
	If you used the constructor method above, you can now retrieve user information from your database within any of your controller methods with: $user = $this→user→get_user(1). The $user variable now contains a database result set object (for the user with id = 1) that can be passed to your View.
	If you passed the entire $user variable to your View, you can access specific user data elements within your View in the form $user→name.

	$this allows objects to reference themselves. You can use it in same way you'd use the object name outside the class

	

	- properties/attributes
 
	properties are class specific variables

	-> accesses contained properties and methods of a given object

	getproperty, setproperty..

 
- **views** Views are files that contain the display information for your application. The purpose of views is to keep this information separate from your application logic for easy reusability and cleaner code.

	Views themselves can contain code used for displaying the data you pass into them. For example, looping through an array of product information and display each one on a new table row. Views are still PHP files so you can use any code you normally would. However, you should try to keep your views as "dumb" as possible and retrieve all data you need in your controllers, then pass it to the view.
	View objects will typically be created inside a Controller using the View::factory method.



Mention variables, private vs public, exceptions, session...





----------


## 4. Routes ##


Routes are used to determine the controller and action for a requested URI. Every route generates a regular expression which is used to match a URI and a route. Routes may also contain keys which can be used to set the controller, action, and parameters.



Example:

    Route::set('default', '(<controller>(/<action>(/<id>)))')
	->defaults(array(
	'controller' => 'Welcome',
	'action' => 'index',
	));


More info: [http://kohanaframework.org/3.3/guide/kohana/routing](http://kohanaframework.org/3.3/guide/kohana/routing)



----------

## 5. Config + templates ##

- bootstrap - The bootstrap is located at application/bootstrap.php. It is responsible for setting up the Kohana environment and executing the main response. It is responsible for the flow of your application. Normally bootstrap contains the routing information but we have moved it out to avoid mess. hence      `require('routes.php');`


- lmu externals

	the externals are a set of files common to all applications, like templates, css etc.. 

- database 

	here you set your database connection details and edb environments

- constants

	
- doctrine- orm 



----------
## 6. SEMS WALKTHROUGH ##


examples:






1. Login + authentication process

[https://sems-pre.leedsmet.ac.uk/login](https://sems-pre.leedsmet.ac.uk/login)


    Route::set('login', 'login')
    ->defaults(array(
    'controller' => 'auth',
    'action' => 'login',
    ));


the url is loading the auth controller and doing the login action

The auth controller is in the file under the controller folder called auth.ph


    class Controller_Auth extends  Controller_Template_AppTemplate {

    protected $user;
    protected $auth_required = FALSE;
        
    
  The login bit of the url tells the controller to load the login action below:
  
    public function action_login() {

        $session = Session::instance();


this bit loads the view which displays the actual loginform

        $view = View::factory('auth/loginform');

  more details here
`if there are variables posted
`   

        if ($this->request->method() == 'POST') {



            // gather the inputs
            $username = $this->request->post('username');
            $password = $this->request->post('password');

            // authentication
            // check LDAP
            // check if we need to skip auth
            if (Kohana::$config->load('application.skip_auth')) {


put variable in new instance of object USER


                $this->user = new Model_User(
                        array('userid' => $username, 'is_admin' => 'Y'), true
                );

                $session->set('username', $this->user->userid);
                $session->set('authed', true);
                $this->request->redirect('/search');
            }

            $view->set('username', $username);
            $ldap = new Library_LDAP();


            if (!$ldap->auth($username, $password)) {
                // not authed
                $message = "Login failed";

                // persist the username, but not the password
                $this->template->errors[] = 'Login details incorrect';
                $this->template->content .= $view;
                //  throw new Exception_User_Badlogin('Login details incorrect');
                return true;
            }
            $message = "Login pass";
            // find the user
            $this->user = Model_User::find($username);






            if (!$this->user) {
                // can't find them, they dont' have access
                $message = "Login failed";


                $this->template->errors[] = 'You dont have access to the system';
                $this->template->content .= $view;
                //throw new Exception_User_Noaccess('You dont have access to the system');
                return true;
            }
            // persist the username, but not the password


            $is_admin = $this->user->__get('is_admin');

            IF ($is_admin = 'Y') {
                $session->set('is_admin', true);
            }
            // set session 

            $session->set('username', $this->user->userid);


            $session->set('authed', true);



            $this->request->redirect('/search');

            // send the message to the view
            $view->set('message', $message);
        }

        $this->template->content .= $view;
    }
