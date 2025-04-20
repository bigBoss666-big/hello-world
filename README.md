# hello-world
练手


#上面是原有内容，此处是进行第一次编辑在分支README
#hello-world2
//import org.slf4j.LoggerFactory;
//
//import java.util.ArrayList;
//import java.util.List;
//import java.util.logging.Logger;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.scheduling.config.FixedRateTask;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@Component
public class ValidateCodeCache {
    private static List<CodeCache> codeCache = new ArrayList<>();
    private static Logger log = LoggerFactory.getLogger(ValidateCodeCache.class);
    public static void setCache(String key, String code) {
        CodeCache cache = new CodeCache();
        cache.setKey(key);
        cache.setCode(code);
        cache.setTimestamp(System.currentTimeMillis());
        codeCache.add(cache);
        log.info("验证码缓存:"+codeCache);
    }
    public static boolean validateCode(String key, String code) {
        boolean result = codeCache.stream().anyMatch(cache -> cache.getKey().equals(key) && cache.getCode().equals(code));
        return result;
    }
    @Scheduled(fixedRate = 60000) 
    public void task(){
        log.info("============开始清理验证码缓存，验证码集合长度：" + codeCache.size() + "============");
        List<CodeCache> codelist = codeCache.stream().filter(cache -> {
            long timestamp = cache.getTimestamp();
            long duration = System.currentTimeMillis() - timestamp;
            return duration > 60000;
        }).collect(Collectors.toList());
        codeCache.removeAll(codelist);
        if (codeCache.size() > 1024){
            codeCache.clear();
        }
        log.info("============清理验证码缓存结束，验证码集合长度：" + codeCache.size() + "============");
    }
}
//这是清理验证码缓存的代码
