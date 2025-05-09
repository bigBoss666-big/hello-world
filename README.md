![image](https://github.com/user-attachments/assets/9c9c60a2-0c6b-4b73-be5b-92592382545c)# hello-world
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


//这是获取当前用户信息的代码
package com.example.utils;

import cn.hutool.core.date.DateUtil;
import cn.hutool.core.util.ObjectUtil;
import com.auth0.jwt.JWT;
import com.auth0.jwt.algorithms.Algorithm;
import com.example.common.Constants;
import com.example.common.enums.RoleEnum;
import com.example.entity.Account;
import com.example.service.AdminService;
import com.example.service.StaffService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.annotation.PostConstruct;
import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import java.util.Date;

/**
 * Token工具类
 */
@Component
public class TokenUtils {

    private static final Logger log = LoggerFactory.getLogger(TokenUtils.class);

    private static AdminService staticAdminService;
    private static StaffService staticStaffService;
    @Resource
    AdminService adminService;

    @Resource
    StaffService staffService;

    // 将静态变量赋值
    @PostConstruct
    public void setUserService() {
        staticAdminService = adminService;
        staticStaffService = staffService;
    }

    /**
     * 生成token
     */
    public static String createToken(String data, String sign) {
        return JWT.create().withAudience(data) // 将 userId-role 保存到 token 里面,作为载荷
                .withExpiresAt(DateUtil.offsetHour(new Date(), 2)) // 2小时后token过期
                .sign(Algorithm.HMAC256(sign)); // 以 password 作为 token 的密钥
    }

    /**
     * 获取当前登录的用户信息
     */
    public static Account getCurrentUser() {
        try {
            HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
            String token = request.getHeader(Constants.TOKEN);
            if (ObjectUtil.isNotEmpty(token)) {
                String userRole = JWT.decode(token).getAudience().get(0);
                String userId = userRole.split("-")[0];  // 获取用户id
                String role = userRole.split("-")[1];    // 获取角色
                if (RoleEnum.ADMIN.name().equals(role)) {
                    return staticAdminService.selectById(Integer.valueOf(userId));
                } else if (RoleEnum.STAFF.name().equals(role)){
                    return staticStaffService.selectById(Integer.valueOf(userId));
                }
            }
        } catch (Exception e) {
            log.error("获取当前用户信息出错", e);
        }
        return new Account();  // 返回空的账号对象
    }
}

//标识用户的辅助类
package com.example.utils;

public class CodeCache {
    private String key;//前端传来的uuid，唯一标识
    private String code; //验证码
    private long timestamp; //时间戳
    public String getKey() {
        return key;
    }
    public void setKey(String key) {
        this.key = key;
    }
    public String getCode() {
        return code;
    }
    public void setCode(String code) {
        this.code = code;
    }
    public long getTimestamp() {
        return timestamp;
    }
    public void setTimestamp(long timestamp) {
        this.timestamp = timestamp;
    }
    @Override
    public String toString() {
        return "CodeCache{" +
                "key='" + key + '\'' +
                ", code='" + code + '\'' +
                ", timestamp=" + timestamp +
                '}';
    }
}




//日志用到的辅助类代码

package com.example.utils;

import javax.servlet.http.HttpServletRequest;

public class IpUtils {

    //因为做登录日志，需要获取用户ip，所以添加这个辅助类
    //一般情况下可以从远程标识中找到请求头的信息
    public static String getIpAddr(HttpServletRequest request) {
        String ip = request.getHeader("x-forwarded-for");
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("X-Forwarded-For");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("WL-Proxy-Client-IP");
        }
        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("X-Real-IP");
        }

        if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return "0:0:0:0:0:0:0:1".equals(ip) ? "127.0.0.1" : ip;
    }

}


