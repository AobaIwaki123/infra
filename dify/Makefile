NS ?= dify

ns:
	kubectl create ns $(NS)

secret:
	kubectl apply -f secret.yml -n $(NS)

up:
	argocd app create dify --file app.yml

down:
	argocd app delete dify -n argocd
