FROM node:24.14.0 as builder

WORKDIR /build_dir

COPY src/portal /build_dir
COPY api/v2.0/swagger.yaml /build_dir/swagger.yaml
RUN npm install --unsafe-perm
RUN npm run postinstall
RUN npm run generate-build-timestamp
RUN node --max_old_space_size=2048 'node_modules/@angular/cli/bin/ng' build --configuration production
RUN npm install js-yaml@4.1.0 \
    && node -e "const yaml = require('js-yaml'); const fs = require('fs'); const swagger = yaml.load(fs.readFileSync('swagger.yaml', 'utf8')); fs.writeFileSync('dist/swagger.json', JSON.stringify(swagger));"
RUN cd app-swagger-ui && npm install --unsafe-perm
RUN cd app-swagger-ui && npm run build

FROM nginx:1.17



COPY --from=builder /build_dir/dist /usr/share/nginx/html
COPY --from=builder /build_dir/app-swagger-ui/dist /usr/share/nginx/html
COPY src/portal/docker-build/nginx.conf /etc/nginx/nginx.conf

EXPOSE 8080
VOLUME /var/cache/nginx /var/log/nginx /run

STOPSIGNAL SIGQUIT

HEALTHCHECK CMD curl --fail -s http://127.0.0.1:8080 || exit 1
USER nginx
CMD ["nginx", "-g", "daemon off;"]
