GroovyInterceptor：
通过实现GroovyInterceptor，可以在运行时拦截和控制特定方法调用。比如，可以拦截 Runtime.exec() 和 ProcessBuilder 的方法调用，阻止执行操作系统命令。
```java
import org.codehaus.groovy.runtime.powerassert.GroovyInterceptor

class MyInterceptor extends GroovyInterceptor {
    @Override
    Object onMethodCall(MethodCall call) {
        if (call.method.equals("exec")) {
            throw new SecurityException("OS command execution is not allowed!")
        }
        return super.onMethodCall(call)
    }
}

def interceptor = new MyInterceptor()
interceptor.install()

// 执行脚本
// 示例代码，运行 exec() 会触发异常
```

import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import org.codehaus.groovy.runtime.powerassert.GroovyInterceptor;

public class SafeScriptWithInterceptor {
    public static void main(String[] args) throws Exception {
        // 设置一个自定义拦截器来限制命令执行
        GroovyInterceptor interceptor = new GroovyInterceptor() {
            @Override
            public Object onMethodCall(MethodCall call) {
                if ("exec".equals(call.getMethodAsString())) {
                    throw new SecurityException("禁止执行操作系统命令！");
                }
                return super.onMethodCall(call);
            }
        };
        interceptor.install();  // 安装拦截器

        // 使用 ScriptEngine 执行脚本
        ScriptEngineManager manager = new ScriptEngineManager();
        ScriptEngine engine = manager.getEngineByName("groovy");

        try {
            engine.eval("Runtime.getRuntime().exec('ls')");  // 应该触发拦截器的异常
        } catch (Exception e) {
            System.out.println("安全拦截生效: " + e.getMessage());
        } finally {
            interceptor.uninstall();  // 执行完毕后卸载拦截器
        }
    }
}


SecureASTCustomizer：
使用SecureASTCustomizer在编译时对脚本进行限制。例如，可以禁止导入 java.lang.Runtime 和 java.lang.ProcessBuilder，这样任何涉及操作系统命令的代码将无法编译通过。
需要增加
secureCustomizer.setIndirectImportCheckEnabled(true);
import org.codehaus.groovy.control.CompilerConfiguration
import org.codehaus.groovy.control.customizers.SecureASTCustomizer

def config = new CompilerConfiguration()
def secureCustomizer = new SecureASTCustomizer()

secureCustomizer.addImportsBlacklist(['java.lang.Runtime', 'java.lang.ProcessBuilder'])
config.addCompilationCustomizers(secureCustomizer)

def shell = new GroovyShell(config)

// 运行受限的Groovy脚本
// 任何使用 Runtime.exec() 或 ProcessBuilder 的代码会触发错误

import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;
import org.codehaus.groovy.control.CompilerConfiguration;
import org.codehaus.groovy.control.customizers.SecureASTCustomizer;
import org.codehaus.groovy.jsr223.GroovyScriptEngineFactory;

public class SafeScriptEngineExample {
    public static void main(String[] args) {
        // 创建ScriptEngineManager
        ScriptEngineManager manager = new ScriptEngineManager();

        // 创建Groovy的CompilerConfiguration并配置SecureASTCustomizer
        CompilerConfiguration config = new CompilerConfiguration();
        SecureASTCustomizer secureCustomizer = new SecureASTCustomizer();
        
        // 添加禁止导入的类
        secureCustomizer.addImportsBlacklist("java.lang.Runtime", "java.lang.ProcessBuilder");

        // 添加SecureASTCustomizer到CompilerConfiguration
        config.addCompilationCustomizers(secureCustomizer);

        // 使用GroovyScriptEngineFactory创建带安全配置的ScriptEngine
        GroovyScriptEngineFactory factory = new GroovyScriptEngineFactory();
        ScriptEngine engine = factory.getScriptEngine(config);

        // 将配置好的ScriptEngine添加到ScriptEngineManager
        manager.registerEngineName("groovy", engine);

        // 测试安全配置的ScriptEngine
        try {
            engine.eval("Runtime.getRuntime().exec('ls')");  // 此处应触发安全异常
        } catch (ScriptException e) {
            System.out.println("安全限制生效，无法执行操作系统命令: " + e.getMessage());
        }
    }
}


