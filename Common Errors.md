COMMON ERRORS
=====================

##### Table of Contents

1. [Mismatched anonymous define() modules ...](#Mismatched_anonymous_define_modules)
1. [Load timeout for modules: ...](#Load_timeout_for_modules)
1. [Error evaluating module ...](#Error_evaluating_module)
1. [모듈 이름 ... Context를 위해 아직 로드되지 않았습니다.](#Module_name_has_not_been_loaded_yet_for_context)
1. [Invalid require call](#Invalid_require_call)
1. [No define call for ...](#No_define_call_for)
1. [Script error](#Script_error)
1. [No matching script interactive for ...](#No_matching_script_interactive_for)
1. [Path is not supported: ...](#Path_is_not_supported)
1. [Cannot use preserveLicenseComments and generateSourceMaps together](#Cannot_use_preserveLicenseComments_and_generateSourceMaps_together)
1. [importScripts failed for ...](#importScripts_failed_for)

This page lists errors that are generated by RequireJS. If the following information does not fix the problem, you can ask on the RequireJS list or open an issue. In either case it is best to have an example or detailed explanation of the problem, hopefully with steps to reproduce.

<a name="Mismatched_anonymous_define_modules">
## MISMATCHED ANONYMOUS DEFINE() MODULES ...

If you manually code a script tag in HTML to load a script with an anonymous define() call, this error can occur.
Also seen if you manually code a script tag in HTML to load a script that has a few named modules, but then try to load an anonymous module that ends up having the same name as one of the named modules in the script loaded by the manually coded script tag.
Finally, if you use the loader plugins or anonymous modules (modules that call define() with no string ID) but do not use the RequireJS optimizer to combine files together, this error can occur. The optimizer knows how to name anonymous modules correctly so that they can be combined with other modules in an optimized file.
To avoid the error:
Be sure to load all scripts that call define() via the RequireJS API. Do not manually code script tags in HTML to load scripts that have define() calls in them.
If you manually code an HTML script tag, be sure it only includes named modules, and that an anonymous module that will have the same name as one of the modules in that file is not loaded.
If the problem is the use of loader plugins or anonymous modules but the RequireJS optimizer is not used for file bundling, use the RequireJS optimizer.

<a name="Load_timeout_for_modules">
## LOAD TIMEOUT FOR MODULES: ...

Likely causes and fixes:
There was a script error in one of the listed modules. If there is no script error in the browser's error console, and if you are using Firebug, try loading the page in another browser like Chrome or Safari. Sometimes script errors do not show up in Firebug.
The path configuration for a module is incorrect. Check the "Net" or "Network" tab in the browser's developer tools to see if there was a 404 for an URL that would map to the module name. Make sure the script file is in the right place. In some cases you may need to use the paths configuration to fix the URL resolution for the script.
The paths config was used to set two module IDs to the same file, and that file only has one anonymous module in it. If module IDs "something" and "lib/something" are both configured to point to the same "scripts/libs/something.js" file, and something.js only has one anonymous module in it, this kind of timeout error can occur. The fix is to make sure all module ID references use the same ID (either choose "something" or "lib/something" for all references), or use map config.

<a name="Error_evaluating_module">
## ERROR EVALUATING MODULE ...

An error occured when the define() function was called for the module given in the error message. It is an error with the code logic inside the define function. The error could happen inside a require callback.
In Firefox and WebKit browsers, a line number and file name will be indicated in the error. It can be used to locate the source of the problem. Better isolation of the error can be done by using a debugger to place a breakpoint in the file that contains the error.

<a name="Module_name_has_not_been_loaded_yet_for_context">
## MODULE NAME ... HAS NOT BEEN LOADED YET FOR CONTEXT: ...(모듈 이름 ... Context를 위해 아직 로드되지 않았습니다.)

This occurs when there is a require('name') call, but the 'name' module has not been loaded yet.

이것은 require('name')을 호출할 경우 아직 'name' 모듈이 로드되지 않았을 때 발생됩니다.

If the error message includes Use require([]), then it was a top-level require call (not a require call inside a define() call) that should be using the async, callback version of require to load the code:

만약 require([])를 사용할 때 에러 메시지가 나온다면, 코드를 로드하는데 require의 비동기 콜백 버전을 사용하는 top-level require 요청입니다.
```javascript
//If this code is not in a define call,
//DO NOT use require('foo'), but use the async
//callback version:
require(['foo'], function (foo) {
    //foo is now loaded.
});
```
If you are using the simplified define wrapper, make sure you have require as the first argument to the definition function:

만약 당신이 단순화된 define 랩퍼를 사용한다면, 정의한 함수의 첫번째 인수로 요청했는지 확인하십시오.
```javascript
define(function (require) {
    var namedModule = require('name');
});
```
If you are listing dependencies in the dependency array, make sure that require and name are in the dependency array:

당신이 dependency 배열에서 dependency를 나열하는 경우, dependency 배열에 'require', 'name'이 있는지 확인하십시오.
```javascript
define(['require', 'name'], function (require) {
    var namedModule = require('name');
});
```
In particular, the following will not work:

특히, 다음은 작동하지 않습니다:
```javascript
//THIS WILL FAIL
define(['require'], function (require) {
    var namedModule = require('name');
});
```
This fails because requirejs needs to be sure to load and execute all dependencies before calling the factory function above. If a dependency array is given to define(), then requirejs assumes that all dependencies are listed in that array, and it will not scan the factory function for other dependencies. So, either do not pass in the dependency array, or if using the dependency array, list all the dependencies in it.

requirejs는 위 factory 함수를 호출하기 전에 모든 dependency를 로드하고 실행해야하는 필요가 있기 때문에 실패합니다. dependency 배열이 define()에 주어졌다면, requirejs는 모든 dependency가 배열에 열거되었다고 간주하고, 다른 dependency를 위한 factory 함수를 검사하지 않습니다. 그래서 dependency 배열에 통하지 않거나, 만약 dependency 배열을 사용한다면, 그 안에 모든 dependency가 있어야 합니다.

If part of a require() callback, all the dependencies need to be listed in the array:

require() 콜백의 부분이라면, 모든 dependency는 배열안에 열거될 필요가 있습니다.
```javascript
require(['require', 'name'], function (require) {
    var namedModule = require('name');
});
```
Be sure that require('name') only occurs inside a define() definition function or a require() callback function, never in the global space by its own.

require('name')은 define() 정의 함수안이거나 require() 콜백 함수 안에 나타나지만, 자기 자신에 의해서 글로벌 공간에 나타나지 않는 것을 명심해야 합니다.

In the RequreJS 1.0.x releases, there is a bug with having a space between the require and parens in WebKit browsers when using the simplified CommonJS wrapping (no dependency array):

RequireJS 1.0.x 릴리즈에서 간단한 CommonJS 랩핑을 사용할 경우 WebKit 브라우저에서 require와 괄호 사이에 공백이 있는 버그가 있습니다(dependency 배열에는 없습니다).
```javascript
define(function (require) {
    //Notice the space between require and the arguments.
    var namedModule = require ('name');
});
```
The workaround is to just remove the space. This is fixed in the 2.0 code, and may be backported to the 1.0.x series if a 1.0.9 release is done.

해결방법은 단지 공간을 제거하는 것입니다. 2.0 코드에서는 고쳤졌고, 만약 1.0.x이 1.0.9에서 릴리즈가 끝난다면, 백포트(소급 수정)될 것입니다.

<a name="Invalid_require_call">
## INVALID REQUIRE CALL

This occurs when there is a call like:
require('dependency', function (dependency) {});
Asynchronously loading dependencies should use an array to list the dependencies:
require(['dependency'], function (dependency) {});

<a name="No_define_call_for">
## NO DEFINE CALL FOR ...

This occurs when enforceDefine is set to true, and a script that is loaded either:
Did not call define() to declare a module.
Or was part of a shim config that specified a string exports property that can be checked to verify loading, and that check failed.
Or was part of a shim config that did not set a string value for the exports config option.
Or, if the error shows up only in IE and not in other browsers (which may generate a Script error, the script probably:
Threw a JavaScript syntax/evaluation error.
Or there was a 404 error in IE where the script failed to load.
Those IE behaviors result in IE's quirks in detecting script errors.
To fix it:
If the module calls define(), make sure the define call was reached by debugging in a script debugger.
If part of a shim config, make sure the shim config's exports check is correct.
If in IE, check for an HTTP 404 error or a JavaScript sytnax error by using a script debugger.

<a name="Script_error">
## SCRIPT ERROR

This occurs when the script.onerror function is triggered in a browser. This usually means there is a JavaScript syntax error or other execution problem running the script. To fix it, examine the script that generated the error in a script debugger.
This error may not show up in IE, just other browsers, and instead, in IE you may see the No define call for ... error when you see "Script error". This is due to IE's quirks in detecting script errors.

<a name="No_matching_script_interactive_for">
## NO MATCHING SCRIPT INTERACTIVE FOR ...

This error only shows up in some IE browsers. Most likely caused by loading a script that calls define() but was loaded in a plain script tag or via some other call, like an eval() of a JavaScript string.
To avoid the error, be sure to load all scripts that call define via the RequireJS API.

<a name="Path_is_not_supported">
## PATH IS NOT SUPPORTED: ...

This error occurs when the optimizer encounters a path to a module or script which is a network path. The optimizer only allows building with local resources. To fix it:
Make sure you reference the network dependency as a module name, not as a full URL, so that it can be mapped to a different during the build:
```javascript
//DO NOT DO THIS
require(['http://some.domain.dom/path/to/dependency.js'],
function (dependency) {});

//Rather, do this:
require.config({
    paths: {
        'dependency': 'http://some.domain.dom/path/to/dependency'
    }
});

require(['dependency'], function (dependency) {});
```
If you want to include this dependency in the built/optimized file, download the JS file and in the build profile for the optimizer, put in a paths config that points to that local file.
If you want to exclude that file from being included, and just need to map "dependency" for the build (otherwise it will not build), then use the special "empty:" paths config:
```javascript
//Inside the build profile
{
    paths: {
        'dependency': 'empty:'
    }
}
```

<a name="Cannot_use_preserveLicenseComments_and_generateSourceMaps_together">
## CANNOT USE PRESERVELICENSECOMMENTS AND GENERATESOURCEMAPS TOGETHER

In the r.js optimizer, preserveLicenseComments works as a pre- and post-processing step on a JS file. Various kinds of license comments are found, pulled out of the JS source, then that modified source is passed to the minifier. When the minifier is done, the comments are added to the top of the file by the r.js optimizer.
However, for the minifier to accurately construct a source map, the minified source cannot be modified in any way, so preserveLicenseComments is incompatible with generateSourceMaps. generateSourceMaps was introduced in version 2.1.2 of the optimizer.
The default for the optimizer is for preserveLicenseComments to be true. So if using generateSourceMaps, then explicitly set preserveLicenseComments to false. If you want to preserve some license comments, you can manually modify the license comments in the JS source to use the JSDoc-style @license comment. See "Annotating JavaScript for the Closure Compiler" for more information. That same format works for UglifyJS2.

<a name="importScripts_failed_for">
## IMPORTSCRIPTS FAILED FOR ...

When RequireJS is used in a Web Worker, importScripts is used to load modules. If that call failed for some reason, this error is generated.


Latest Release: 2.1.14  
Open source: new BSD or MIT licensed  
web design by Andy Chung © 2011-2014  