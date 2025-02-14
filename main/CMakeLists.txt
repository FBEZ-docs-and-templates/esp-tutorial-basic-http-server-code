idf_component_register(SRCS "basic_http_server.c"
                    PRIV_REQUIRES nvs_flash esp_wifi esp_http_server
                    INCLUDE_DIRS ".")
