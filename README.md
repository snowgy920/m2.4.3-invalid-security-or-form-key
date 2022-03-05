# m2.4.3-invalid-security-or-form-key
You can apply this patch as well.

diff --git a/App/Action/Plugin/Authentication.php b/App/Action/Plugin/Authentication.php
index 8227966..19795a5 100644
--- a/App/Action/Plugin/Authentication.php
+++ b/App/Action/Plugin/Authentication.php
@@ -225,13 +225,24 @@ class Authentication
 
         // Checks, whether secret key is required for admin access or request uri is explicitly set
         if ($this->_url->useSecretKey()) {
-            $requestParts = explode('/', trim($request->getRequestUri(), '/'), 3);
-            $baseUrlPath = trim(parse_url($this->backendUrl->getBaseUrl(), PHP_URL_PATH), '/');
-            $routeIndex = empty($baseUrlPath) ? 0 : 1;
-            $requestUri = $this->_url->getUrl($requestParts[$routeIndex]);
+            if (strpos(trim($request->getRequestUri(),'/'), $request->getFrontName()) === 0) {
+                // request starts with admin router frontname, get parts 1-2-3 for redirect
+                $requestParts = explode('/', trim($request->getRequestUri(), '/'.$request->getFrontName()), 4);
+            } else {
+                // request doesnt start with admin router frontname, get first three parts for redirect
+                $requestParts = explode('/', trim($request->getRequestUri(), '/'), 3);
+            }
+
+            $requestParams = $request->getParams();
+
+            //remove key and form key params
+            unset($requestParams['key'],$requestParams['form_key']);
+            
+            $requestUri = $this->_url->getUrl(implode('/',$requestParts), $requestParams);
+
         } elseif ($request) {
             $requestUri = $request->getRequestUri();
-        }
+        } 
 
         if (!$requestUri) {
             return false;
