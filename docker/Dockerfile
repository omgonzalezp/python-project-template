# ==============================================================================
# ETAPA 1: BUILD (Compilación y Instalación Robusta)
# ==============================================================================
FROM python:3.11-slim AS builder

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# Instala dependencias del sistema necesarias
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    wget \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp/app

# Copia los requisitos e instala
COPY requirements.txt /tmp/app/
RUN pip install --no-cache-dir -r requirements.txt

# INSTALACIÓN DE TAILWINDCSS CLI STANDALONE 
RUN wget https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-linux-x64 -O /usr/local/bin/tailwindcss \
    && chmod +x /usr/local/bin/tailwindcss


# ==============================================================================
# ETAPA 2: PROD (Imagen Ligera para Ejecución)
# ==============================================================================
FROM python:3.11-alpine AS prod 

RUN apk add --no-cache libpq

# Directorio de Trabajo Final: /src (Raíz del Código)
WORKDIR /src

# Copia la CLI de Tailwind y el entorno virtual ya construido
COPY --from=builder /usr/local/bin/tailwindcss /usr/local/bin/tailwindcss
COPY --from=builder /usr/local/lib/python3.11/site-packages/ /usr/local/lib/python3.11/site-packages/

# Copia el código fuente base (será reemplazado por el bind mount)
COPY src /src/

RUN adduser -D myuser
USER myuser

# Comando de producción (Gunicorn, asumiendo un archivo wsgi.py en la raíz de src)
CMD ["gunicorn", "wsgi:application", "--bind", "0.0.0.0:8000"]


# ==============================================================================
# ETAPA 3: DEV (Imagen Robusta para Desarrollo)
# ==============================================================================
FROM builder AS dev 

# Directorio de Trabajo Final: /src
WORKDIR /src

# Copia el código base
COPY src /src/

# Comando de desarrollo (Ejecuta un script principal para prueba/live-reload)
CMD ["python", "main.py"]