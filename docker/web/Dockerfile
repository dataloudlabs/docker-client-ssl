FROM nginx

COPY nginx.conf /etc/nginx/nginx.conf
COPY confs/*.conf /etc/nginx/conf.d/

COPY auth/ca.crt /etc/nginx/certs/ca.crt
COPY auth/server.crt /etc/nginx/certs/server.crt
COPY auth/server.key /etc/nginx/certs/server.key

COPY /auth/dhparam.pem /etc/nginx/certs/dhparam.pem

