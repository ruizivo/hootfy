# FROM node:20-alpine


# RUN apk update
# RUN apk add nginx
# # RUN adduser -D -g 'www' www
# # RUN chown -R www:www /var/lib/nginx
# # RUN chown -R www:www /www

# WORKDIR /app

# COPY package*.json ./

# RUN npm install

# COPY . .

# #EXPOSE 3000
# #CMD ["npm", "run", "start"]


# # Copia frontend buildado para o nginx
# COPY /src/front /usr/share/nginx/html

# # # Copia config custom do nginx
# # COPY nginx.conf /etc/nginx/conf.d/default.conf

# # Copia sua config nginx personalizada
# COPY nginx.conf /etc/nginx/nginx.conf

# # Entrypoint para iniciar o backend + nginx
# COPY docker-entrypoint.sh /docker-entrypoint.sh
# RUN chmod +x /docker-entrypoint.sh

# EXPOSE 80
# CMD ["/docker-entrypoint.sh"]





# Estágio de build do frontend
FROM node:20-alpine AS frontend-builder

WORKDIR /app/hootfy-front

# Copiar apenas arquivos necessários para instalar dependências
COPY src/hootfy-front/package*.json ./

# Instalar dependências
RUN npm ci

# Copiar o restante dos arquivos do frontend
COPY src/hootfy-front/ ./

# Construir o aplicativo frontend
RUN npm run build

# Estágio de build do backend
FROM node:20-alpine AS backend-builder

WORKDIR /app

# Copiar apenas arquivos necessários para instalar dependências
COPY package*.json ./

# Instalar dependências de produção
RUN npm ci --only=production && npm cache clean --force

# Copiar o código fonte do backend
COPY config ./config
COPY src/core ./src/core
COPY src/storage ./src/storage
COPY src/utils ./src/utils
COPY src/web ./src/web

# Estágio final - imagem enxuta para produção
FROM alpine:3.18

# # Instalar Node.js e Nginx (versões mínimas necessárias)
# RUN apk add --no-cache nodejs nginx && \
#     rm -rf /var/cache/apk/*

# Instalar Node.js, Nginx e dependências necessárias para o Puppeteer
RUN apk add --no-cache \
    nodejs \
    nginx \
    chromium \
    nss \
    freetype \
    harfbuzz \
    ca-certificates \
    ttf-freefont \
    font-noto-emoji \
    && rm -rf /var/cache/apk/*

# Criar diretório para o aplicativo
WORKDIR /app

# Copiar dependências e código do backend do estágio builder
COPY --from=backend-builder /app ./
# COPY --from=backend-builder /app/src ./src
# COPY --from=backend-builder /app/src ./src

# Copiar os arquivos do frontend compilado para o diretório do Nginx
COPY --from=frontend-builder /app/hootfy-front/dist/ /usr/share/nginx/html/

# Configuração do Nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Script de entrada
COPY docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh

# Expor apenas a porta HTTP
EXPOSE 80

# Executar o script de entrada ao iniciar o container
CMD ["/docker-entrypoint.sh"]


