NS = n8n

cp:
	cp secret.example.yml postgres-secret.yaml

ns:
	kubectl create ns $(NS)

secret:
	kubectl apply -f postgres-secret.yaml -n $(NS)

up:
	argocd app create $(NS) --file app.yml

down:
	argocd app delete $(NS) -y
