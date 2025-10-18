NS = flowise

cp:
	cp secret.example.yml secret.yml

ns:
	kubectl create ns $(NS)

secret:
	kubectl apply -f secret.yml -n $(NS)

up:
	argocd app create $(NS) --file app.yml

down:
	argocd app delete $(NS) -y
