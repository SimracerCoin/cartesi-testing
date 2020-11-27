# Testing - Processing iRacing race results file in a Cartesi Maching

This project contains code used to test the Descartes SDK, documented in https://cartesi.io/docs/, by processing a iRacing race results file in a Cartesi machine.

## Getting Started

### Requirements

- docker
- docker-compose
- node 12.x
- yarn
- truffle

### Environment

Download and run the Descartes SDK Environment ready-to-use artifact by executing:

```
wget https://github.com/cartesi/descartes-tutorials/releases/download/v0.2.0/descartes-env-0.2.0.tar.gz
tar -xzvf descartes-env-0.2.0.tar.gz
```
Then, start it up by running:

```
cd descartes-env
docker-compose up
```

### Cartesi Playground

Clone this project and use the cartesi/playground Docker image, making sure to map the current directory:

```
docker run -it --rm \
  -e USER=$(id -u -n) \
  -e GROUP=$(id -g -n) \
  -e UID=$(id -u) \
  -e GID=$(id -g) \
  -v `pwd`:/home/$(id -u -n) \
  -w /home/$(id -u -n) \
  cartesi/playground:0.1.1 /bin/bash
```
  
Pack the processresult.sh script, that does the processing of the iRacing results file provided, for usage within a Cartesi Machine. For that build an ext2 file-system with the script file by running:

```
mkdir ext2
cp processresult.sh ext2
```

Still inside the playground, use the genext2fs tool to generate the file-system with those contents:

```
genext2fs -b 1024 -d ext2 processresult.ext2
```
Use the truncate tool to pad the results.csv file to 8K for usage with the Cartesi machine:


```
truncate -s 8K results.csv
```

Still within the playground, execute the following command to process the iRacing race results file in a cartesi machine:

```
cartesi-machine \
  --flash-drive="label:processresult,filename:processresult.ext2" \
  --flash-drive="label:input,length:1<<13,filename:results.csv" \
  --flash-drive="label:output,length:1<<12,filename:output.raw,shared" \
  -- $'cd /mnt/processresult ; dd if=$(flashdrive input) of=results.raw ; ./processresult.sh results.raw > $(flashdrive output)'
```

The result of the processing will be stored in the output.raw file
