　　首先看ProxyHook
```java
public abstract class ProxyHook extends Hook implements InvocationHandler {
protected Object mOldObj;
public ProxyHook(Context hostContext) {
    super(hostContext);
}
public void setOldObj(Object oldObj) {
    this.mOldObj = oldObj;
}
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        if (!isEnable()) {
            return method.invoke(mOldObj, args);
        }
        HookedMethodHandler hookedMethodHandler = mHookHandles.getHookedMethodHandler(method);
        if (hookedMethodHandler != null) {
            return hookedMethodHandler.doHookInner(mOldObj, method, args);
        }
        return method.invoke(mOldObj, args);
    } catch (InvocationTargetException e) {
        Throwable cause = e.getTargetException();
        if (cause != null && MyProxy.isMethodDeclaredThrowable(method, cause)) {
            throw cause;
        } else if (cause != null) {
            RuntimeException runtimeException = !TextUtils.isEmpty(cause.getMessage()) ? new RuntimeException(cause.getMessage()) : new RuntimeException();
            runtimeException.initCause(cause);
            throw runtimeException;
        } else {
            RuntimeException runtimeException = !TextUtils.isEmpty(e.getMessage()) ? new RuntimeException(e.getMessage()) : new RuntimeException();
            runtimeException.initCause(e);
            throw runtimeException;
        }
    } catch (IllegalArgumentException e) {
        try {
            StringBuilder sb = new StringBuilder();
            sb.append(" DROIDPLUGIN{");
            if (method != null) {
                sb.append("method[").append(method.toString()).append("]");
            } else {
                sb.append("method[").append("NULL").append("]");
            }
            if (args != null) {
                sb.append("args[").append(Arrays.toString(args)).append("]");
            } else {
                sb.append("args[").append("NULL").append("]");
            }
            sb.append("}");

            String message = e.getMessage() + sb.toString();
            throw new IllegalArgumentException(message, e);
        } catch (Throwable e1) {
            throw e;
        }
    } catch (Throwable e) {
        if (MyProxy.isMethodDeclaredThrowable(method, e)) {
            throw e;
        } else {
            RuntimeException runtimeException = !TextUtils.isEmpty(e.getMessage()) ? new RuntimeException(e.getMessage()) : new RuntimeException();
            runtimeException.initCause(e);
            throw runtimeException;
        }
    }
}
```