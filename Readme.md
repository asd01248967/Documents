# ngx_http_geoip2_module 

1. 安裝 libmaxminddb

2. 下載 [nginx](http://nginx.org/download/) 來源碼，並且版本要與實際運行的伺服器一致

3. 下載 [ngx_http_geoip2_module](https://github.com/leev/ngx_http_geoip2_module) 來源碼

4. 個別解壓縮後，進入 nginx 來源碼的目錄下

5. 執行以下命令， --add-dynamic-module 等於 ngx_http_geoip2_module 目錄
    

    $ ./configure --add-dynamic-module=/path/to/ngx_http_geoip2_module --with-compat
    $ make
    

6. 編譯完後， Module 位於 objs/ 下的 ngx_http_geoip2_module.so 

7. 將 ngx_http_geoip2_module.so 上傳至伺服器，並在 nginx main block 設定檔下新增以下設定
    

    load_module modules/ngx_http_geoip2_module.so;
    

8. 在 nginx http block 設定檔下新增以下設定
    

    ### Geo IP ###
    geoip2 /usr/share/GeoIP/GeoLite2-Country.mmdb {
        $geoip2_data_country_code country iso_code;
    }
    map $geoip2_data_country_code $allowed_country {
        default yes;    # 預設允許全部國家存取
        CN no;          # 拒絕存取的國家代碼
    }
    geo $remote_addr $ip_whitelist {
        default no;     # 預設沒有白 IP
        1.1.1.1/32 yes; # 允許的白 IP
    }
    ##############
    

9. 在需要阻擋的 nginx server block 下新增以下設定
    

    ### 如果 $ip_whitelist = yes ，則停止當前的 rewrite 處理
    if ($ip_whitelist = yes) {
        break;
    }

    if ($allowed_country = no) {
        return 403;
    }