build:
	# protoc -I. --go_out=plugins=grpc:$(PWD) proto/consignment/consignment.proto
	protoc -I. --go_out=plugins=micro:$(PWD) proto/consignment/consignment.proto

	GOOS=linux GOARCH=amd64 go build 
	docker build -t consignment-service .

run:
	docker run -p 50051:50051 -d consignment-service:latest