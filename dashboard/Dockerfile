FROM node:22-alpine
WORKDIR /app
COPY dashboard/package*.json ./dashboard/
RUN cd dashboard && npm ci --no-audit --no-fund
COPY dashboard ./dashboard
COPY shared ./shared
COPY ros2/zone.json ./ros2/zone.json
WORKDIR /app/dashboard
RUN npm run build
CMD ["npm", "run", "preview", "--", "--host", "0.0.0.0", "--port", "5173"]
