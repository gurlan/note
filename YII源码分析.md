# 前言

本文主要分析Yii2应用的启动、运行的过程，主要包括以下三部分：入口脚本、启动应用、运行应用。

# 入口脚本

```php
<?php

// 定义全局变量
defined('YII_DEBUG') or define('YII_DEBUG', true);
defined('YII_ENV') or define('YII_ENV', 'dev');

// composer自动加载代码机制，可参考 https://segmentfault.com/a/1190000010788354
require __DIR__ . '/../vendor/autoload.php';

// 1.引入工具类Yii
// 2.注册自动加载函数
// 3.生成依赖注入中使用到的容器
require __DIR__ . '/../vendor/yiisoft/yii2/Yii.php';

// 加载应用配置文件
$config = require __DIR__ . '/../config/web.php';

//生成应用实例并运行
(new yii\web\Application($config))->run();
```

# 启动应用

**1.分析：new yii\web\Application($config)**
主要就是执行父类构造函数（代码位置：vendor\yiisoft\yii2\base\Application.php）

```php
public function __construct($config = [])
    {
  		  // 将Yii::$app 保存为当前对象
        Yii::$app = $this;
  			
  			// 见分析2
        static::setInstance($this);

  		  // 设置当前应用状态
        $this->state = self::STATE_BEGIN;

        // 进行一些预处理（根据$config配置应用）
        // 详细代码位置：yii\base\Application::preInit(&$config) 见分析3
        $this->preInit($config);
				
  			//注册ErrorHandler，这样一旦抛出了异常或错误，就会由其负责处理
 			  //代码位置：yii\base\Application::registerErrorHandler(&$config) 见分析4 
        $this->registerErrorHandler($config);
	
  			// 根据$config配置Application
  			// 然后执行yii\web\Application::init()（实际上是执行yii\base\Application::init()）见分析5
        Component::__construct($config);
    }
```

**2.分析：yii\base\Module::setInstance($instance)**

```php
  	/**
     * Sets the currently requested instance of this module class.
     * @param Module|null $instance the currently requested instance of this module class.
     * If it is `null`, the instance of the calling class will be removed, if any.
     */
	public static function setInstance($instance)
    {
        if ($instance === null) {
            unset(Yii::$app->loadedModules[get_called_class()]);
        } else {
            // 将Application实例本身记录为“已加载模块”
            Yii::$app->loadedModules[get_class($instance)] = $instance;
        }
    }
```

**3.分析：yii\base\Application::preInit(&$config)**

```php
		/**
     * Pre-initializes the application.
     * This method is called at the beginning of the application constructor.
     * It initializes several important application properties.
     * If you override this method, please make sure you call the parent implementation.
     * @param array $config the application configuration
     * @throws InvalidConfigException if either [[id]] or [[basePath]] configuration is missing.
     */
    public function preInit(&$config)
    {
      	// 1.判断$config中是否有配置项‘id’，‘basePath’（必须有,否则抛异常）
        if (!isset($config['id'])) {
            throw new InvalidConfigException('The "id" configuration for the Application is required.');
        }
        if (isset($config['basePath'])) {
            $this->setBasePath($config['basePath']);
            unset($config['basePath']);
        } else {
            throw new InvalidConfigException('The "basePath" configuration for the Application is required.');
        }
				//	2.设置别名：@vendor，@runtime
        if (isset($config['vendorPath'])) {
            $this->setVendorPath($config['vendorPath']);
            unset($config['vendorPath']);
        } else {
            // set "@vendor"
            $this->getVendorPath();
        }
      	
        if (isset($config['runtimePath'])) {
            $this->setRuntimePath($config['runtimePath']);
            unset($config['runtimePath']);
        } else {
            // set "@runtime"
            $this->getRuntimePath();
        }
				
      	// 3.设置时间区域（timeZone）
        if (isset($config['timeZone'])) {
            $this->setTimeZone($config['timeZone']);
            unset($config['timeZone']);
        } elseif (!ini_get('date.timezone')) {
            $this->setTimeZone('UTC');
        }
				
      	// 4.自定义配置容器（Yii::$container）的属性（由这里我们知道可以自定义配置容器）
        if (isset($config['container'])) {
            $this->setContainer($config['container']);
            unset($config['container']);
        }
				
      	// 5.合并核心组件配置到自定义组件配置：数组$config['components']
      	//（核心组件有哪些参考：yii\web\Application::coreComponents()）
        //（注意：这个方法中$config使用了引用，所以合并$config['components']可以改变$config原来的值）
        // merge core components with custom components
        foreach ($this->coreComponents() as $id => $component) {
            if (!isset($config['components'][$id])) {
                $config['components'][$id] = $component;
            } elseif (is_array($config['components'][$id]) && !isset($config['components'][$id]['class'])) {
                $config['components'][$id]['class'] = $component['class'];
            }
        }
    }

```

**4.分析：yii\base\Application::registerErrorHandler(&$config)**

```php
  	/**
     * Registers the errorHandler component as a PHP error handler.
     * @param array $config application config
     */
    protected function registerErrorHandler(&$config)
    {
        if (YII_ENABLE_ERROR_HANDLER) {
            if (!isset($config['components']['errorHandler']['class'])) {
                echo "Error: no errorHandler component is configured.\n";
                exit(1);
            }
            $this->set('errorHandler', $config['components']['errorHandler']);
            unset($config['components']['errorHandler']);
            $this->getErrorHandler()->register();
        }
    }
	

    // 默认config文件关于error handler的配置，这里的配置会合并到Yii2本身对核心组件的定义中去（），核心组件的定义在yii\web\Application::coreComponents()
    'components' => [
            'errorHandler' => [
                'errorAction' => 'site/error',
            ],
    ],

		// yii\base\ErrorHandler::register()

 		/**
     * Register this error handler.
     * @since 2.0.32 this will not do anything if the error handler was already registered
     */
    public function register()
    {
        if (!$this->_registered) {
            // 该选项设置是否将错误信息作为输出的一部分显示到屏幕，
    			  // 或者对用户隐藏而不显示。
            ini_set('display_errors', false);
          	// 当抛出异常且没有被catch的时候,由yii\base\ErrorHandler::handleException（）处理
            set_exception_handler([$this, 'handleException']);
            if (defined('HHVM_VERSION')) {
                set_error_handler([$this, 'handleHhvmError']);
            } else {
              	// 当出现error的时候由yii\base\ErrorHandler::handleError（）处理
                set_error_handler([$this, 'handleError']);
            }
            if ($this->memoryReserveSize > 0) {
                $this->_memoryReserve = str_repeat('x', $this->memoryReserveSize);
            }
            // 当出现fatal error的时候由yii\base\ErrorHandler::handleFatalError（）处理
            register_shutdown_function([$this, 'handleFatalError']);
            $this->_registered = true;
        }
    }
一个值得注意的地方:
在handleException()、handleError()、handleFatalError()中会直接或间接调用yii\web\ErrorHandle::renderException(),而在这个函数里面，有以下这一行代码
$result = Yii::$app->runAction($this->errorAction);
回顾上面config文件中对errorAction这个属性的定义，我们知道可以通过自定义一个action用于在异常或错误时显示自定义的出错页面
例如yii2中默认使用’site/error’这个action来处理
```

**5.分析：Component::__construct($config)**
（代码实际位置：/vendor/yiisoft/yii2/base/BaseObject.php）

```php
		/**
     * Constructor.
     *
     * The default implementation does two things:
     *
     * - Initializes the object with the given configuration `$config`.
     * - Call [[init()]].
     *
     * If this method is overridden in a child class, it is recommended that
     *
     * - the last parameter of the constructor is a configuration array, like `$config` here.
     * - call the parent implementation at the end of the constructor.
     *
     * @param array $config name-value pairs that will be used to initialize the object properties
     */
    public function __construct($config = [])
    {
        if (!empty($config)) {
          	// 见下面的详细分析
            Yii::configure($this, $config);
        }
        // 跳转到yii\base\Application::init()，见下面的分析
        $this->init();
    }

		
		 /**
     * Configures an object with the initial property values.
     * @param object $object the object to be configured
     * @param array $properties the property initial values given in terms of name-value pairs.
     * @return object the object itself
     */
		//Yii::configure($object, $properties)分析
    //实际上就是为Application的属性赋值
    //注意：这里会触发一些魔术方法,间接调用了：
    //yii\web\Application::setHomeUrl()
    //yii\di\ServiceLocator::setComponents()（这个值得注意，详细参考本文后续“5.1分析”）
    //yii\base\Module::setModules()
    public static function configure($object, $properties)
    {
        foreach ($properties as $name => $value) {
            $object->$name = $value;
        }

        return $object;
    }
		
    /**
     * {@inheritdoc}
     */
		// yii\base\Application::init()分析
    public function init()
    {
      	// 设置当前应用状态
        $this->state = self::STATE_INIT;
      	// 见下面的bootstrap()方法
        $this->bootstrap();
    }
	
		 /**
     * Initializes extensions and executes bootstrap components.
     * This method is called by [[init()]] after the application has been fully configured.
     * If you override this method, make sure you also call the parent implementation.
     */
    protected function bootstrap()
    {
        if ($this->extensions === null) {
           // @vendor/yiisoft/extensions.php是一些关于第三方扩展的配置，当用composer require安装第三扩展的时候就会将新的扩展的相关信息记录到该文件，这样我们就可以在代码中调用了
            $file = Yii::getAlias('@vendor/yiisoft/extensions.php');
            $this->extensions = is_file($file) ? include $file : [];
        }
        foreach ($this->extensions as $extension) {
            if (!empty($extension['alias'])) {
                foreach ($extension['alias'] as $name => $path) {
                    Yii::setAlias($name, $path);
                }
            }
            // 进行一些必要的预启动
            if (isset($extension['bootstrap'])) {
                $component = Yii::createObject($extension['bootstrap']);
                if ($component instanceof BootstrapInterface) {
                    Yii::debug('Bootstrap with ' . get_class($component) . '::bootstrap()', __METHOD__);
                    $component->bootstrap($this);
                } else {
                    Yii::debug('Bootstrap with ' . get_class($component), __METHOD__);
                }
            }
        }
				// 预启动，通常会包括‘log’组件，‘debug’模块和‘gii’模块（参考配置文件）
        foreach ($this->bootstrap as $mixed) {
            $component = null;
            if ($mixed instanceof \Closure) {
                Yii::debug('Bootstrap with Closure', __METHOD__);
                if (!$component = call_user_func($mixed, $this)) {
                    continue;
                }
            } elseif (is_string($mixed)) {
                if ($this->has($mixed)) {
                    $component = $this->get($mixed);
                } elseif ($this->hasModule($mixed)) {
                    $component = $this->getModule($mixed);
                } elseif (strpos($mixed, '\\') === false) {
                    throw new InvalidConfigException("Unknown bootstrapping component ID: $mixed");
                }
            }

            if (!isset($component)) {
                $component = Yii::createObject($mixed);
            }

            if ($component instanceof BootstrapInterface) {
                Yii::debug('Bootstrap with ' . get_class($component) . '::bootstrap()', __METHOD__);
                $component->bootstrap($this);
            } else {
                Yii::debug('Bootstrap with ' . get_class($component), __METHOD__);
            }
        }
    }

```

在完成了上面的流程之后，应用就算启动成功了，可以开始运行，处理请求了。

# 运行应用

**1.分析：yii\base\Application::run()**
在上面的构造函数执行完后，开始运行应用。即下面这行代码的run()部分
(new yii\web\Application($config))->run();//（实际上执行的是yii\base\Application::run()）

```php
 /**
     * Runs the application.
     * This is the main entrance of an application.
     * @return int the exit status (0 means normal, non-zero values mean abnormal)
     */
    public function run()
    {
        try {
            $this->state = self::STATE_BEFORE_REQUEST;
          	// 触发事件’beforeRequest’,依次执行该事件的handler，
            $this->trigger(self::EVENT_BEFORE_REQUEST);

            $this->state = self::STATE_HANDLING_REQUEST;
          	//处理请求，这里的返回值是yii\web\Response实例
            //handleRequest(),详细参考本文后续“分析2”
            $response = $this->handleRequest($this->getRequest());

            $this->state = self::STATE_AFTER_REQUEST;
          	// 触发事件’afterRequest’,依次执行该事件的handler
            $this->trigger(self::EVENT_AFTER_REQUEST);
		
            $this->state = self::STATE_SENDING_RESPONSE;
            // 发送响应,详细参考本文后续“分析3”
            $response->send();

            $this->state = self::STATE_END;

            return $response->exitStatus;
        } catch (ExitException $e) {
          	// 结束运行
            $this->end($e->statusCode, isset($response) ? $response : null);
            return $e->statusCode;
        }
    }
```

**2.分析： yii\web\Application::handleRequest()**

```php
 /**
     * Handles the specified request.
     * @param Request $request the request to be handled
     * @return Response the resulting response
     * @throws NotFoundHttpException if the requested route is invalid
     */
    public function handleRequest($request)
    {
        if (empty($this->catchAll)) {
            try {
              	// 从请求中解析路由和参数，这里会调用urlManager组件来处理
                list($route, $params) = $request->resolve();
            } catch (UrlNormalizerRedirectException $e) {
                $url = $e->url;
                if (is_array($url)) {
                    if (isset($url[0])) {
                        // ensure the route is absolute
                        $url[0] = '/' . ltrim($url[0], '/');
                    }
                    $url += $request->getQueryParams();
                }
								 // 当解析请求出现异常时进行重定向
                return $this->getResponse()->redirect(Url::to($url, $e->scheme), $e->statusCode);
            }
        } else {
          	// ’catchAll’参数可以在配置文件中自定义，可用于在项目需要临时下线维护时给出一个统一的访问路由
            $route = $this->catchAll[0];
            $params = $this->catchAll;
            unset($params[0]);
        }
        try {
            Yii::debug("Route requested: '$route'", __METHOD__);
          	// 记录下当前请求的route
            $this->requestedRoute = $route;
          	// 执行路由相对应的action（yii\base\Module::runAction()详细参考本文后续“分析3”）
            $result = $this->runAction($route, $params);
            if ($result instanceof Response) {
                return $result;
            }
						// 如果action的返回结果不是Response的实例，则将结果封装到Response实例的data属性中
            $response = $this->getResponse();
            if ($result !== null) {
                $response->data = $result;
            }

            return $response;
        } catch (InvalidRouteException $e) {
            throw new NotFoundHttpException(Yii::t('yii', 'Page not found.'), $e->getCode(), $e);
        }
    }
```

**3.分析： yii\base\Module::runAction()**

```php
public function runAction($route, $params = [])
{
    //根据路由创建Controller实例（详细参考本文后续“分析4”）
    $parts = $this->createController($route);
    if (is_array($parts)) {
        /* @var $controller Controller */
      	// 解析出控制器和方法
        list($controller, $actionID) = $parts;
        $oldController = Yii::$app->controller;
        // 设置当前的Controller实例
        Yii::$app->controller = $controller;
        // 执行action（yii\base\Controller::runAction()详细参考本文后续“分析5”）
        $result = $controller->runAction($actionID, $params);
        if ($oldController !== null) {
            //可以看做是栈
            Yii::$app->controller = $oldController;
        }

        return $result;
    }

    $id = $this->getUniqueId();
    throw new InvalidRouteException('Unable to resolve the request "' . ($id === '' ? $route : $id . '/' . $route) . '".');
}
```

**4.分析： yii\base\Module::createController()**

```PHP
		 /**
     * Creates a controller instance based on the given route.
     *
     * The route should be relative to this module. The method implements the following algorithm
     * to resolve the given route:
     *
     * 1. If the route is empty, use [[defaultRoute]];
     * 2. If the first segment of the route is found in [[controllerMap]], create a controller
     *    based on the corresponding configuration found in [[controllerMap]];
     * 3. If the first segment of the route is a valid module ID as declared in [[modules]],
     *    call the module's `createController()` with the rest part of the route;
     * 4. The given route is in the format of `abc/def/xyz`. Try either `abc\DefController`
     *    or `abc\def\XyzController` class within the [[controllerNamespace|controller namespace]].
     *
     * If any of the above steps resolves into a controller, it is returned together with the rest
     * part of the route which will be treated as the action ID. Otherwise, `false` will be returned.
     *
     * @param string $route the route consisting of module, controller and action IDs.
     * @return array|bool If the controller is created successfully, it will be returned together
     * with the requested action ID. Otherwise `false` will be returned.
     * @throws InvalidConfigException if the controller class and its file do not match.
     */
    public function createController($route)
    {
      	 // 如果route为空则设置route为默认路由，这个可以在配置文件中自定义
        if ($route === '') {
            $route = $this->defaultRoute;
        }

        // double slashes or leading/ending slashes may cause substr problem
        $route = trim($route, '/');
        if (strpos($route, '//') !== false) {
            return false;
        }

        if (strpos($route, '/') !== false) {
            list($id, $route) = explode('/', $route, 2);
        } else {
            $id = $route;
            $route = '';
        }
			
      	// 优先使用controllerMap，controllerMap可以如下
        // module and controller map take precedence
        if (isset($this->controllerMap[$id])) {
            $controller = Yii::createObject($this->controllerMap[$id], [$id, $this]);
            return [$controller, $route];
        }
      	// 先判断是否存在相应的模块
        $module = $this->getModule($id);
        if ($module !== null) {
          	// 当存在模块时，进行递归
            return $module->createController($route);
        }

        if (($pos = strrpos($route, '/')) !== false) {
            $id .= '/' . substr($route, 0, $pos);
            $route = substr($route, $pos + 1);
        }
				// 最终找到Controller的id
        $controller = $this->createControllerByID($id);
        if ($controller === null && $route !== '') {
            // 详细见下面的代码分析
            $controller = $this->createControllerByID($id . '/' . $route);
            $route = '';
        }
				 // 返回Controller实例和剩下的路由信息
        return $controller === null ? false : [$controller, $route];
    }
		
		/**
     * Creates a controller based on the given controller ID.
     *
     * The controller ID is relative to this module. The controller class
     * should be namespaced under [[controllerNamespace]].
     *
     * Note that this method does not check [[modules]] or [[controllerMap]].
     *
     * @param string $id the controller ID.
     * @return Controller|null the newly created controller instance, or `null` if the controller ID is invalid.
     * @throws InvalidConfigException if the controller class and its file name do not match.
     * This exception is only thrown when in debug mode.
     */
    public function createControllerByID($id)
    {
        $pos = strrpos($id, '/');
        if ($pos === false) {
            $prefix = '';
            $className = $id;
        } else {
            $prefix = substr($id, 0, $pos + 1);
            $className = substr($id, $pos + 1);
        }

        if ($this->isIncorrectClassNameOrPrefix($className, $prefix)) {
            return null;
        }

        $className = preg_replace_callback('%-([a-z0-9_])%i', function ($matches) {
                return ucfirst($matches[1]);
            }, ucfirst($className)) . 'Controller';
        $className = ltrim($this->controllerNamespace . '\\' . str_replace('/', '\\', $prefix) . $className, '\\');
        if (strpos($className, '-') !== false || !class_exists($className)) {
            return null;
        }

        if (is_subclass_of($className, 'yii\base\Controller')) {
          	// 通过依赖注入容器获得Controller实例
            $controller = Yii::createObject($className, [$id, $this]);
            return get_class($controller) === $className ? $controller : null;
        } elseif (YII_DEBUG) {
            throw new InvalidConfigException('Controller class must extend from \\yii\\base\\Controller.');
        }

        return null;
    }
```

**5.分析：yii\base\Controller::runAction()**

```php
/**
     * Runs an action within this controller with the specified action ID and parameters.
     * If the action ID is empty, the method will use [[defaultAction]].
     * @param string $id the ID of the action to be executed.
     * @param array $params the parameters (name-value pairs) to be passed to the action.
     * @return mixed the result of the action.
     * @throws InvalidRouteException if the requested action ID cannot be resolved into an action successfully.
     * @see createAction()
     */
    public function runAction($id, $params = [])
    {
      	// 创建action实例，详细见下面的代码
        $action = $this->createAction($id);
        if ($action === null) {
            throw new InvalidRouteException('Unable to resolve the request: ' . $this->getUniqueId() . '/' . $id);
        }

        Yii::debug('Route to run: ' . $action->getUniqueId(), __METHOD__);

        if (Yii::$app->requestedAction === null) {
            Yii::$app->requestedAction = $action;
        }

        $oldAction = $this->action;
        $this->action = $action;

        $modules = [];
        $runAction = true;

      	// 返回的modules包括该controller当前所在的module，以及该module的所有祖先module（递归直至没有祖先module）
    		// 然后从最初的祖先module开始，依次执行“模块级”的beforeAction()
     		// 如果有beforeAction()没有返回true， 那么会中断后续的执行
        // call beforeAction on modules
        foreach ($this->getModules() as $module) {
            if ($module->beforeAction($action)) {
                array_unshift($modules, $module);
            } else {
                $runAction = false;
                break;
            }
        }

        $result = null;
				// 执行当前控制器的beforeAction，通过后再最终执行action
   		  //（如果前面“模块级beforeAction”没有全部返回true，则这里不会执行）
        if ($runAction && $this->beforeAction($action)) {
            // run the action
          	//代码位置：yii\base\InlineAction::runWithParams()详细参考本文后续
            $result = $action->runWithParams($params);
						// 执行当前Controller的afterAction
            $result = $this->afterAction($action, $result);

            // call afterAction on modules
          	// 从当前模块开始，执行afterAction，直至所有祖先的afterAction
            foreach ($modules as $module) {
                /* @var $module Module */
                $result = $module->afterAction($action, $result);
            }
        }

        if ($oldAction !== null) {
            $this->action = $oldAction;
        }
				// 如果有beforeAction没有通过，那么会返回null
        return $result;
    }
		
		/**
     * Creates an action based on the given action ID.
     * The method first checks if the action ID has been declared in [[actions()]]. If so,
     * it will use the configuration declared there to create the action object.
     * If not, it will look for a controller method whose name is in the format of `actionXyz`
     * where `xyz` is the action ID. If found, an [[InlineAction]] representing that
     * method will be created and returned.
     * @param string $id the action ID.
     * @return Action|null the newly created action instance. Null if the ID doesn't resolve into any action.
     */
		// 根据给定的动作ID创建一个动作。该方法首先检查动作ID是否已在[[actions（）]]中声明。如果是这样，它将使用在那里声明的配置来创建操作对象。如果不是，它将寻找名称为actionXyz格式的控制器方法，其中xyz是动作ID。如果找到，将创建并返回表示该方法的[[InlineAction]]。
    public function createAction($id)
    {
        if ($id === '') {
            $id = $this->defaultAction;
        }

        $actionMap = $this->actions();
        if (isset($actionMap[$id])) {
            return Yii::createObject($actionMap[$id], [$id, $this]);
        }

        if (preg_match('/^(?:[a-z0-9_]+-)*[a-z0-9_]+$/', $id)) {
            $methodName = 'action' . str_replace(' ', '', ucwords(str_replace('-', ' ', $id)));
            if (method_exists($this, $methodName)) {
                $method = new \ReflectionMethod($this, $methodName);
                if ($method->isPublic() && $method->getName() === $methodName) {
                    // InlineAction封装了将要执行的action的相关信息，该类继承自yii\base\Action
                    return new InlineAction($id, $this, $methodName);
                }
            }
        }

        return null;
    }


```

**6.分析： yii\web\Response::send()**

```php
		 /**
     * Sends the response to the client.
     */
    public function send()
    {
        if ($this->isSent) {
            return;
        }
      	// 触发事件
        $this->trigger(self::EVENT_BEFORE_SEND);
      	// 预处理 
        $this->prepare();
      	// 触发事件
        $this->trigger(self::EVENT_AFTER_PREPARE);
      	// 发送http响应的头部，详见下面的代码
        $this->sendHeaders();
      	// 发送http响应的主体，详见下面的代码
        $this->sendContent();
      	// 触发事件
        $this->trigger(self::EVENT_AFTER_SEND);
        $this->isSent = true;
    }

```

# 请求生命周期

![请求生命周期](https://www.yiichina.com/doc/guide/2.0/images/request-lifecycle.png)

1. 用户向[入口脚本](https://www.yiichina.com/doc/guide/2.0/structure-entry-scripts) `web/index.php` 发起请求。
2. 入口脚本加载应用[配置](https://www.yiichina.com/doc/guide/2.0/concept-configurations)并创建一个[应用](https://www.yiichina.com/doc/guide/2.0/structure-applications) 实例去处理请求。
3. 应用通过[请求](https://www.yiichina.com/doc/guide/2.0/runtime-request)组件解析请求的 [路由](https://www.yiichina.com/doc/guide/2.0/runtime-routing)。
4. 应用创建一个[控制器](https://www.yiichina.com/doc/guide/2.0/structure-controllers)实例去处理请求。
5. 控制器创建一个[动作](https://www.yiichina.com/doc/guide/2.0/structure-controllers)实例并针对操作执行过滤器。
6. 如果任何一个过滤器返回失败，则动作取消。
7. 如果所有过滤器都通过，动作将被执行。
8. 动作会加载一个数据模型，或许是来自数据库。
9. 动作会渲染一个视图，把数据模型提供给它。
10. 渲染结果返回给[响应](https://www.yiichina.com/doc/guide/2.0/runtime-responses)组件。
11. 响应组件发送渲染结果给用户浏览器。

# 静态结构

![应用静态结构](https://www.yiichina.com/doc/guide/2.0/images/application-structure.png)

每个应用都有一个入口脚本 `web/index.php`，这是整个应用中唯一可以访问的 PHP 脚本。 入口脚本接受一个 Web 请求并创建[应用](https://www.yiichina.com/doc/guide/2.0/structure-application)实例去处理它。 [应用](https://www.yiichina.com/doc/guide/2.0/structure-applications)在它的[组件](https://www.yiichina.com/doc/guide/2.0/concept-components)辅助下解析请求， 并分派请求至 MVC 元素。[视图](https://www.yiichina.com/doc/guide/2.0/structure-views)使用[小部件](https://www.yiichina.com/doc/guide/2.0/structure-widgets) 去创建复杂和动态的用户界面。