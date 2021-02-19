# Introdução ao Nomad

## Anotações do Talk da HashiCorp disponível no link: https://youtu.be/_k4w_AMZVN8

### O que é um Nomad?

Semelhante ao Kubernetes o Nomad é um Orquestrador. Orquestradores possibilitam que os artefatos produzidos pelos desenvolvedores fiquem disponíveis aos clientes utilziando a infraestrutura. Além disso, eles garantem que os artefatos estejam disponíveis e distribuídos na infra, facilitando o gerenciamento dos recursos. 

Vantagens do Nomad:
-Bom para times menores;
-Múltiplos ambientes e tipos de workload;
-Simplicidade de instalação uso e manutenção.

O Nomad posúi uma arquitetura cliente servidor, onde um binário (agente do Nomad) pode agir como cliente/servidor dentro da infra. Com especialidade na orquestração de workloads, o Nomad possuí foco na simplicidade de deploy, com ou sem containers.

### Jobs

A forma como vc vai descrever os seus workloads, pode ser um serviço (vai ta sempre rodando):

```
job "example" {
  type = "service"
```

Um serviço do tipo batch, que vai rodar e quando terminar a tarefa será finalizado. Ele pode ser parametrizado. 

```
job "example" {
  type = "service"
  
  parameterized {
    payload           = "required"
    meta_required     = ["dispacher_email"]
    meta_operational  = ["pager_email"]
}
```

Colocar Job pra rodar em um local/data horário específico (periódico)

```
job "example" {
  type = "service"
  
 periodic {
  cron = "*/15 * * * * *"
 }
}
```

Job para rodar em todos os clientes (job de manutenção)

```
job "example" {
  type = "system"
```

#### Definindo grupos dentro dos Jobs

Dentro de cada Job vc pode criar um ou mais grupos, o grupo é a unidade que pode ser agendada.

```
job "example" {
  type = "service"
  datacenter = ["dc1"]
  
  group "cache" {
    count = 3 // quantas instâncias do grupo vc quer rodar
    
    //Definindo a fronteira de recursos
    network {
      mode = "bridge"
      port "redis" {}
    }
  }
```

Dentro dos grupos vc define as Tasks (o processo que vai rodar, que os clientes vão rodar em suas máquinas, elas podem ser de vários tipos: docker, java, binário, VM Qemu):

```
job "example" {
 ...
  
  group "cache" {
    ...
    
    tasks "redis"{
      driver = "docker"
      
      config {
        image = "redis:v3.2"
    }
  }
```

Definindo ordem específica de execução com o lifecycle:

```
job "example" {
  task "init" {
    lifecycle {
      hook = "prestart" //  prestart define que essa task roda primeiro
    }
  }
  task "sidecar" {
    lifecycle {
      sidecar = true // roda junto das outras
    }
  }
```

#### Executando os Jobs

Escreva o arquivo .nomad e execute com:

```
nomad run example.nomad
```


