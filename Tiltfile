# -*- mode: Python -*-

# Allow Tilt to use your k3s cluster
allow_k8s_contexts('minikube')

# Load MySQL k8s config
k8s_yaml('local-db/mysql.yaml')
k8s_resource('mysql', port_forwards='3306:3306')

# Backend service
docker_build(
    'backend',
    context='/home/asela/git/chores-backend',
    dockerfile='/home/asela/git/chores-backend/Dockerfile',
    live_update=[
        # All sync steps first
        sync('/home/asela/git/chores-backend/app', '/app/app'),
        sync('/home/asela/git/chores-backend/alembic', '/app/alembic'),
        sync('/home/asela/git/chores-backend/alembic.ini', '/app/alembic.ini'),
        # Then run steps
        run('pip install -r requirements.txt',
            trigger='requirements.txt')
    ]
)

# Load backend k8s config
k8s_yaml('backend/k8s/deployment.yaml')
k8s_resource(
    'backend', 
    port_forwards='8000:8000',
    resource_deps=['mysql']
)

# Frontend service
docker_build(
    'frontend',
    context='/home/asela/git/chores-frontend',
    dockerfile='/home/asela/git/chores-frontend/Dockerfile',
    live_update=[
        # All sync steps first
        sync('/home/asela/git/chores-frontend/src', '/app/src'),
        sync('/home/asela/git/chores-frontend/public', '/app/public'),
        # Then run steps
        run('npm install', trigger='package.json'),
        run('npm run build && cp -r /app/dist/* /usr/share/nginx/html/', 
            trigger=['src', 'public'])
    ]
)

# Load frontend k8s configs
k8s_yaml(['frontend/k8s/deployment.yaml', 'frontend/k8s/nginx-config.yaml'])
k8s_resource(
    'frontend', 
    port_forwards='3000:80',
    resource_deps=['backend'])