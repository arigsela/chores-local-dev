# -*- mode: Python -*-

# Allow Tilt to run in the 'minikube' Kubernetes context
allow_k8s_contexts('minikube')

# Deploy MySQL database
k8s_yaml('local-db/mysql.yaml')
k8s_resource('mysql', port_forwards='3306:3306')

# Backend service
custom_build(
    'backend',
    'docker build --no-cache -t $EXPECTED_REF /home/asela/git/chores-backend',
    ['/home/asela/git/chores-backend'],
    live_update=[
        sync('/home/asela/git/chores-backend/app', '/app/app'),
        sync('/home/asela/git/chores-backend/alembic', '/app/alembic'),
        sync('/home/asela/git/chores-backend/alembic.ini', '/app/alembic.ini'),
        run('pip install -r requirements.txt', trigger='requirements.txt')
    ],
    ignore=['node_modules', '.git']
)

k8s_yaml('backend/k8s/deployment.yaml')
k8s_resource(
    'backend',
    port_forwards='8000:8000',
    resource_deps=['mysql']
)

# Frontend service
custom_build(
    'frontend',
    'docker build --no-cache -t $EXPECTED_REF /home/asela/git/chores-frontend',
    ['/home/asela/git/chores-frontend'],
    live_update=[
        sync('/home/asela/git/chores-frontend/src', '/app/src'),
        sync('/home/asela/git/chores-frontend/public', '/app/public'),
        run('npm install', trigger='package.json'),
        run('npm run build && cp -r /app/dist/* /usr/share/nginx/html/', trigger=['src', 'public'])
    ],
    ignore=['node_modules', '.git']
)

k8s_yaml(['frontend/k8s/deployment.yaml', 'frontend/k8s/nginx-config.yaml'])
k8s_resource(
    'frontend',
    port_forwards='3000:80',
    resource_deps=['backend']
)

# Set trigger mode to automatic
trigger_mode(TRIGGER_MODE_AUTO)
