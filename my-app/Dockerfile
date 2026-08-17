# ==========================================
# Etapa 1: Instalación de dependencias
# ==========================================
FROM node:20-alpine AS deps
# libc6-compat es necesario en Alpine para ciertas dependencias nativas de Node
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Copiamos solo los manifiestos de paquetes para aprovechar la caché de Docker
COPY package.json package-lock.json* ./
RUN npm ci

# ==========================================
# Etapa 2: Construcción (Build)
# ==========================================
FROM node:20-alpine AS builder
WORKDIR /app

# Traemos las dependencias instaladas de la etapa previa
COPY --from=deps /app/node_modules ./node_modules
COPY . .

# Deshabilitamos la telemetría durante el build y compilamos la app
ENV NEXT_TELEMETRY_DISABLED=1
RUN npm run build

# ==========================================
# Etapa 3: Imagen final de Producción (Runner)
# ==========================================
FROM node:20-alpine AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

# Usamos el usuario 'node' que ya viene preconfigurado en la imagen Alpine
USER node

# Copiamos los archivos públicos estáticos
COPY --from=builder --chown=node:node /app/public ./public

# Copiamos los artefactos standalone y los archivos estáticos de Next.js
COPY --from=builder --chown=node:node /app/.next/standalone ./
COPY --from=builder --chown=node:node /app/.next/static ./.next/static

# Puerto por defecto de Next.js
EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

# Ejecutamos el servidor standalone nativo de Next.js sin necesidad de npm
CMD ["node", "server.js"]