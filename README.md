# Infraestructura como código utilizando AWS CloudFormation
![AWS](https://img.shields.io/badge/Amazon_AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)

En este proyecto vamos a lanzar una infraestructura en la nube que sea altamente disponible, escalable y segura usando las buenas prácticas del well-architect framework. 

El despliegue de la arquitectura se hará por capas utilizando el servicio Cloudformation que es un servicio de AWS de infraestructura como código. La arquitectura a desplegar es la que se muestra en la siguiente imagen. 

![arquitectura](img/arquitecturaHA-HS.png)


<hr>

1. Vamos a crear primero los templates usando la extensión .yml

<hr>

En el primer template denominado network.yml se despliegan los servicios relacionados con la infraestructura de red como el servicio de VPC. A continuación se explica las partes del template:

- En Parameters se definel el CIDR del servicio VPC y los CIDR de las diferentes subredes. Por seguridad se tendrán subredes privadas. 

      Parameters:
        VpcCIRD:
          Type: String
          Description: CIDR of the VPC
          Default: 172.16.0.0/16
        
        PublicSubnetACIDR:
          Type: String
          Description: CIDR of public subnet A
          Default: 172.16.1.0/24

        PublicSubnetBCIDR:
          Type: String
          Description: CIDR of public subnet B
          Default: 172.16.2.0/24
        
        PrivateSubnetACIDR:
          Type: String
          Description: CIDR of private subnet A
          Default: 172.16.3.0/24
        
        PrivateSubnetBCIDR:
          Type: String
          Description: CIDR of private subnet B
          Default: 172.16.4.0/24
        
        PrivateSubnetAACIDR:
          Type: String
          Description: CIDR of private subnet AA
          Default: 172.16.5.0/24
        
        PrivateSubnetBBCIDR:
          Type: String
          Description: CIDR of private subnet BB
          Default: 172.16.6.0/24

      - **Network Layer**
        
                  SGbookPublic:
                  Type: AWS::EC2::SecurityGroup
                  Properties:
                    GroupDescription: Allow connection through SSH
                    SecurityGroupIngress:
                      - IpProtocol: tcp
                        FromPort: 22
                        ToPort: 22
                        CidrIp: 0.0.0.0/0
                      - IpProtocol: tcp
                        FromPort: 5000
                        ToPort: 5000
                        CidrIp: 0.0.0.0/0
                    Tags:
                      - Key: Name
                        Value: sg-book-ws
                    VpcId:
                      Fn::ImportValue:
                        !Sub "network-stack-VPCID"
  
- En **Resources** se configuran los diferentes tipos de servicios con sus respectivas propiedades que harán parte del template network.yml y de la arquitectura a implementar. Dentro de los servicios a desplegar se tiene:

  - VPC: configura una red virtual en la nube de AWS
  - Subnets: se divide esa red virtual VPC en subredes pública (los recursos configurados dentro de esta subred son accesibles desde internet) y subredes privadas (no tienen una ruta directa para acceder a internet). 
  - InternetGateway: servicio que permite la conexión a internet a la VPC
  - NateGateway: servicio que permite que los recursos creados en las subredes privadas puedan acceder a internet. 
  - RouteTable: crea tablas de enrutamiento para definir rutas que redirigen el tráfico en las subredes
  - Routes: se definen las rutas que se asocian a las tablas de enrutamiento. 


        Resources:

          #Configure a VPC
          VPCbookStore:
            Type: AWS::EC2::VPC
            Properties:
              CidrBlock: !Ref VpcCIRD
              EnableDnsHostnames: true
              EnableDnsSupport: true
              Tags:
                - Key: Name
                  Value: VPC-Bookstore-ElMundoDeLasLetras

          #Configure the subnets
          PublicSubnetA:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPCbookStore
              AvailabilityZone: us-east-1a
              CidrBlock: !Ref PublicSubnetACIDR
              MapPublicIpOnLaunch: true
              Tags:
                - Key: Name
                  Value: Public-SubnetA
          
          PublicSubnetB:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPCbookStore
              AvailabilityZone: us-east-1b
              CidrBlock: !Ref PublicSubnetBCIDR
              MapPublicIpOnLaunch: true
              Tags:
                - Key: Name
                  Value: Public-SubnetB
          
          PrivateSubnetA:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPCbookStore
              AvailabilityZone: us-east-1a
              CidrBlock: !Ref PrivateSubnetACIDR
              MapPublicIpOnLaunch: false
              Tags:
                - Key: Name
                  Value: PrivateSubnetA
          
          PrivateSubnetB:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPCbookStore
              AvailabilityZone: us-east-1b
              CidrBlock: !Ref PrivateSubnetBCIDR
              MapPublicIpOnLaunch: false
              Tags:
                - Key: Name
                  Value: PrivateSubnetB
          
          PrivateSubnetAA:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPCbookStore
              AvailabilityZone: us-east-1a
              CidrBlock: !Ref PrivateSubnetAACIDR
              MapPublicIpOnLaunch: false
              Tags:
                - Key: Name
                  Value: PrivateSubnetAA
          
          PrivateSubnetBB:
            Type: AWS::EC2::Subnet
            Properties:
              VpcId: !Ref VPCbookStore
              AvailabilityZone: us-east-1b
              CidrBlock: !Ref PrivateSubnetBBCIDR
              MapPublicIpOnLaunch: false
              Tags:
                - Key: Name
                  Value: PrivateSubnetBB

        #Configure the internet gateway

          InternetGateway:
            Type: AWS::EC2::InternetGateway
            Properties:
              Tags:
                - Key: Name
                  Value: igw-bookstore
          
          InternetGatewayAttachment:
            Type: AWS::EC2::VPCGatewayAttachment
            Properties:
              VpcId: !Ref VPCbookStore
              InternetGatewayId: !Ref InternetGateway
          
          PublicRouteTable:
            Type: AWS::EC2::RouteTable
            Properties:
              VpcId: !Ref VPCbookStore
              Tags:
                - Key: Name
                  Value: PublicRouteTable
          
          PublicRoute:
            Type: AWS::EC2::Route
            DependsOn: InternetGatewayAttachment
            Properties:
              RouteTableId: !Ref PublicRouteTable
              DestinationCidrBlock: 0.0.0.0/0
              GatewayId: !Ref InternetGateway
          
          PublicSubnetARouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PublicSubnetA
              RouteTableId: !Ref PublicRouteTable
          
          PublicSubnetBRouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PublicSubnetB
              RouteTableId: !Ref PublicRouteTable

        #Configure two natgateways in public subnet A and public subnet B

          NatGatewayA:
            Type: AWS::EC2::NatGateway
            Properties:
              AllocationId: !GetAtt AipElastic.AllocationId
              SubnetId: !Ref PublicSubnetA
              Tags:
                - Key: Name
                  Value: NatGatewayA
          
          AipElastic:
            Type: AWS::EC2::EIP
            Properties:
              Domain: vpc
              Tags:
                - Key: Name
                  Value: AipElastic
          
          NatGatewayB:
            Type: AWS::EC2::NatGateway
            Properties:
              AllocationId: !GetAtt BipElastic.AllocationId
              SubnetId: !Ref PublicSubnetB
              Tags:
                - Key: Name
                  Value: NatGatewayB
          
          BipElastic:
            Type: AWS::EC2::EIP
            Properties:
              Domain: vpc
              Tags:
                - Key: Name
                  Value: BipElastic
          
          PrivateRouteTableA:
            Type: AWS::EC2::RouteTable
            Properties:
              VpcId: !Ref VPCbookStore
              Tags:
                - Key: Name
                  Value: PrivateRouteTableA
          
          PrivateRouteTableB:
            Type: AWS::EC2::RouteTable
            Properties:
              VpcId: !Ref VPCbookStore
              Tags:
                - Key: Name
                  Value: PrivateRouteTableB
          
          PrivateRouteA:
            Type: AWS::EC2::Route
            Properties:
              RouteTableId: !Ref PrivateRouteTableA
              DestinationCidrBlock: 0.0.0.0/0
              NatGatewayId: !Ref NatGatewayA
          
          PrivateRouteB:
            Type: AWS::EC2::Route
            Properties:
              RouteTableId: !Ref PrivateRouteTableB
              DestinationCidrBlock: 0.0.0.0/0
              NatGatewayId: !Ref NatGatewayB
          
          PrivateSubnetARouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PrivateSubnetA
              RouteTableId: !Ref PrivateRouteTableA
          
          PrivateSubnetBRouteTableAssociation:
            Type: AWS::EC2::SubnetRouteTableAssociation
            Properties:
              SubnetId: !Ref PrivateSubnetB
              RouteTableId: !Ref PrivateRouteTableB

- En **Outputs** se definen unas variables de salida que estarán disponibles al final del despliegue exitoso del stack. Estas variables de salida será utilizadas al momento del despliegue del stack application.yml para configurar servicios dentro de la infraestructura de red. 

        Outputs:
          VPCbookStore:
            Value: !Ref VPCbookStore
            Export:
              Name: !Sub ${AWS::StackName}-VPCID
          
          PublicSubnetA:
            Value: !Ref PublicSubnetA
            Export:
              Name: !Sub ${AWS::StackName}-PublicSubnetA
          
          PublicSubnetB:
            Value: !Ref PublicSubnetB
            Export:
              Name: !Sub ${AWS::StackName}-PublicSubnetB
          
          PrivateSubnetA:
            Value: !Ref PrivateSubnetA
            Export:
              Name: !Sub ${AWS::StackName}-PrivateSubnetA
          
          PrivateSubnetB:
            Value: !Ref PrivateSubnetB
            Export:
              Name: !Sub ${AWS::StackName}-PrivateSubnetB
          
          PrivateSubnetAA:
            Value: !Ref PrivateSubnetAA
            Export:
              Name: !Sub ${AWS::StackName}-PrivateSubnetAA
          
          PrivateSubnetBB:
            Value: !Ref PrivateSubnetBB
            Export:
              Name: !Sub ${AWS::StackName}-PrivateSubnetBB

<hr>

En el segundo template denominado application.yml se configuran los servicios relacionados con el servidor web y la base de datos. 

- En esta parte del template se configuran los diferentes grupos de seguridad (Security Group) que se van a asociar a la instancia (Bastion Host) que se configura en la subred pública, al servidor web que corre en las subredes privadas, al balanceador de carga y a la base de datos. Dependiendo del grupo de seguridad se definen unas reglas (firewall) de permiso. 


        SGbookPublic:
            Type: AWS::EC2::SecurityGroup
            Properties:
              GroupDescription: Allow connection through SSH
              SecurityGroupIngress:
                - IpProtocol: tcp
                  FromPort: 22
                  ToPort: 22
                  CidrIp: 0.0.0.0/0        
              Tags:
                - Key: Name
                  Value: sg-book-ws
              VpcId:
                Fn::ImportValue:
                  !Sub "network-stack-VPCID"
                

          #Create the SG for instance in subnet private
          SGbookPrivate:
            Type: AWS::EC2::SecurityGroup
            Properties:
              GroupDescription: Allow connection through SSH and HTTP for instance un subnet private
              SecurityGroupIngress:
                - IpProtocol: tcp
                  FromPort: 22
                  ToPort: 22
                  SourceSecurityGroupId: !GetAtt SGbookPublic.GroupId
                - IpProtocol: tcp
                  FromPort: 5000
                  ToPort: 5000
                  SourceSecurityGroupId: !GetAtt SGalb.GroupId
              Tags:
                - Key: Name
                  Value: sg-book-private
              VpcId:
                Fn::ImportValue:
                  !Sub "network-stack-VPCID"

          #Create the SG for ALB
          SGalb:
            Type: AWS::EC2::SecurityGroup
            Properties:
              GroupDescription: Allow connection through HTTP
              SecurityGroupIngress:
                - IpProtocol: tcp
                  FromPort: 80
                  ToPort: 80
                  CidrIp: 0.0.0.0/0     
              Tags:
                - Key: Name
                  Value: sg-alb
              VpcId:
                Fn::ImportValue:
                  !Sub "network-stack-VPCID"


          #Create the SG for DB
          SGdb:
            Type: AWS::EC2::SecurityGroup
            Properties:
              GroupDescription: Allow connection through SSH and MYSQL for DB in subnet private
              SecurityGroupIngress:
                - IpProtocol: tcp
                  FromPort: 22
                  ToPort: 22
                  SourceSecurityGroupId: !GetAtt SGbookPrivate.GroupId
                - IpProtocol: tcp
                  FromPort: 3306
                  ToPort: 3306
                  SourceSecurityGroupId: !GetAtt SGbookPrivate.GroupId
              Tags:
                - Key: Name
                  Value: sg-db
              VpcId:
                Fn::ImportValue:
                  !Sub "network-stack-VPCID"


- En esta parte del template se configura:

  - Bastion Host: en el template se denomina como **bookWSpublic** y es una instancia EC2 que sirve para acceder a través de SSH a los recursos de la subred privada. Se configuran sus diferentes parámetros como tipo de instancia, red, zona de disponibilidad y el User Data que descarga archivos e instala diferente paquetes mientra la instancia se está iniciando por primera vez. En un bucket de S3 de mi cuenta tengo los archivos python-db-ssm.zip y BookDbDump.sql. El archivo BookDbDump.sql ya está listo para migrar la base de datos al servicio de AWS RDS, migración que se hace desde el User Data cuando el servicio AWS RDS esté disponible. 
  

        #Create the instance
        bookWSpublic:
          Type: AWS::EC2::Instance
          DependsOn:
            - DBbook
          Properties:
            AvailabilityZone: us-east-1a
            ImageId: ami-0bb84b8ffd87024d8
            InstanceType: t2.micro
            IamInstanceProfile: EC2s3
            KeyName: iotTest
            NetworkInterfaces:
              - AssociatePublicIpAddress: true
                DeviceIndex: 0
                SubnetId:
                  Fn::ImportValue:
                    !Sub "network-stack-PublicSubnetA"
                GroupSet:
                  - Ref: SGbookPublic       
            Tags:
              - Key: Name
                Value: Bastion-Host
            UserData: 
              Fn::Base64: !Sub | 
                #!/bin/bash
                sudo dnf install -y python3.9-pip
                pip install virtualenv
                sudo dnf install -y mariadb105-server
                sudo service mariadb start
                sudo chkconfig mariadb on
                pip install flask
                pip install mysql-connector-python
                pip install boto3
                wget https://jav-bucket-web.s3.amazonaws.com/python-db-ssm.zip          
                wget https://jav-bucket-web.s3.amazonaws.com/databases.zip
                wget https://jav-bucket-web.s3.amazonaws.com/BookDbDump.sql
                sleep 2
                sudo unzip python-db-ssm.zip
                sudo unzip databases.zip 
                sudo mv python-db-ssm databases /home/ec2-user
                wget https://jav-bucket-web.s3.amazonaws.com/bookapp.service
                sudo mv bookapp.service /etc/systemd/system
                DB_PASSWORD=$(aws ssm get-parameter --name "/book/password" --with-decryption --query "Parameter.Value" --output text --region "us-east-1")
                mysql -u root -p$DB_PASSWORD --host ${DBbook.Endpoint.Address} < BookDbDump.sql
                sudo systemctl daemon-reload
                sudo systemctl start bookapp
                sudo systemctl enable bookapp  
        
- En esta parte se configuran los servicios necesarios para configurar el Application Load Balancer (Balanceador de carga) y el Auto Scaling Group. Estos servicios permiten que la arquitectura a desplegar sea altamente disponible y escalable. Los servicios a configurar son:

  - LaunchTemplate: denonimado en el template LaunchTemplateBook que configura una plantilla que utiliza el servicio Auto Scaling Group para lanzar el servidor web en las subredes privadas de acuerdo a los parámetros definidos en el LaunchTemplateBook.  
  - ElasticLoadBalancingV2::LoadBalancer: balanceador de carga que se configura de tipo Application Load Balancer y se despliega en las subredes públicas en zonas de disponibilidad diferente. 
  - TargetGroup: denominado en el template TGelb, configura el destino al cual el Application Load Balancer redirige el tráfico. Para el presente template son las instancias EC2. 
  - Listener: denominado en el template ListenerALB, define el puerto a través del cual escucha el Application Load Balancer y cómo se configuran las reglas del Target Group. 
  - AutoScalingGroup: denominado ASGbook, configura el servicio Auto Scaling Group donde se determina la cantidad de instancias para atender el tráfico promedio y cuántas instancia se incrementa en caso que ocurra un pico en el tráfico. 
  - ScalingPolicy: denominado en el template ScalingPolicyASG, determina el tipo de politica de escalamiento y la métrica a utilizar en el Auto Scaling Group. Para este template se utiliza como métrica el uso del CPU, en caso que el uso promedio de la CPU esté por encima del 25%, se despliegan dos intancias más para tener en total 4 instancias.  

        ##Create a launch Template
        LaunchTemplateBook:
          Type: AWS::EC2::LaunchTemplate
          DependsOn:
            - DBbook
          Properties:
            LaunchTemplateData:
              IamInstanceProfile: 
                Name: EC2s3   
              ImageId: ami-0bb84b8ffd87024d8
              KeyName: iotTest
              InstanceType: t2.micro
              SecurityGroupIds: 
                - Ref: SGbookPrivate  
              UserData: 
                Fn::Base64: !Sub | 
                  #!/bin/bash
                  sudo dnf install -y python3.9-pip
                  pip install virtualenv
                  sudo dnf install -y mariadb105-server
                  sudo service mariadb start
                  sudo chkconfig mariadb on
                  pip install flask
                  pip install mysql-connector-python
                  pip install boto3
                  wget https://jav-bucket-web.s3.amazonaws.com/python-db-ssm.zip          
                  wget https://jav-bucket-web.s3.amazonaws.com/databases.zip
                  wget https://jav-bucket-web.s3.amazonaws.com/BookDbDump.sql
                  sleep 2
                  sudo unzip python-db-ssm.zip
                  sudo unzip databases.zip 
                  sudo mv python-db-ssm databases /home/ec2-user
                  wget https://jav-bucket-web.s3.amazonaws.com/bookapp.service
                  sudo mv bookapp.service /etc/systemd/system
                  DB_PASSWORD=$(aws ssm get-parameter --name "/book/password" --with-decryption --query "Parameter.Value" --output text --region "us-east-1")
                  mysql -u root -p$DB_PASSWORD --host ${DBbook.Endpoint.Address} < BookDbDump.sql
                  sudo systemctl daemon-reload
                  sudo systemctl start bookapp
                  sudo systemctl enable bookapp 
                  pip install virtualenv
                  sudo dnf install -y mariadb105-server
                  sudo service mariadb start
                  sudo chkconfig mariadb on
                  pip install flask
                  pip install mysql-connector-python
                  pip install boto3
                  wget https://jav-bucket-web.s3.amazonaws.com/python-db-ssm.zip
                  wget https://jav-bucket-web.s3.amazonaws.com/databases.zip
                  sudo unzip python-db-ssm.zip
                  sudo unzip databases.zip 
                  sudo mv python-db-ssm databases /home/ec2-user
                  wget https://jav-bucket-web.s3.amazonaws.com/bookapp.service
                  sudo mv bookapp.service /etc/systemd/system
                  sudo systemctl daemon-reload
                  sudo systemctl start bookapp
                  sudo systemctl enable bookapp
            LaunchTemplateName: lt-book 

        ##Create the Application Load Balancer
        ALBbook:
          Type: AWS::ElasticLoadBalancingV2::LoadBalancer
          Properties:
            Name: alb-book
            Scheme: internet-facing
            SecurityGroups: 
              - Ref: SGalb
            Subnets: 
              - Fn::ImportValue: !Sub "network-stack-PublicSubnetA"
              - Fn::ImportValue: !Sub "network-stack-PublicSubnetB"
            Type: application

        #Create the target group
        TGelb:
          Type: AWS::ElasticLoadBalancingV2::TargetGroup
          Properties:
            HealthCheckEnabled: true
            HealthCheckPath: /health
            Name: tg-wsbook
            Port: 5000
            Protocol: HTTP
            TargetType: instance
            VpcId: 
              Fn::ImportValue:
                !Sub "network-stack-VPCID"

        #Create a listener for ALB
        ListenerALB:
          Type: AWS::ElasticLoadBalancingV2::Listener
          Properties:
            LoadBalancerArn: 
              Ref: ALBbook
            Port: 80
            Protocol: HTTP 
            DefaultActions:
              - Type: forward 
                TargetGroupArn: 
                  Ref: TGelb  
        
        #Create the Auto Scaling group
        ASGbook:
          Type: AWS::AutoScaling::AutoScalingGroup
          Properties:
            AutoScalingGroupName: asg-book
            DesiredCapacity: 2
            MaxSize: 4
            MinSize: 2
            LaunchTemplate: 
              LaunchTemplateId: 
                Ref: LaunchTemplateBook          
              Version: !GetAtt LaunchTemplateBook.LatestVersionNumber
            TargetGroupARNs: 
              - Ref: TGelb
            VPCZoneIdentifier: 
              - Fn::ImportValue: !Sub "network-stack-PrivateSubnetA"
              - Fn::ImportValue: !Sub "network-stack-PrivateSubnetB"

        #Create the Scaling policy
        ScalingPolicyASG:
          Type: AWS::AutoScaling::ScalingPolicy
          Properties:
            AutoScalingGroupName: 
              Ref: ASGbook
            PolicyType: TargetTrackingScaling
            TargetTrackingConfiguration: 
              PredefinedMetricSpecification: 
                PredefinedMetricType: ASGAverageCPUUtilization
              TargetValue: 25

- En esta parte se configuran los servicios relacionados con la configuración del servicio AWS RDS donde se depliega una base de datos relacional MariaDB. 

  - DBbook: se configura la instancia que despliega la base de datos, el tipo de motor de bases de datos, el usuario y la contraseña. Por seguridad la contraseña de la base de datos se configura en el Servicio Parameter Store de AWS System Manager. 
  - DBbookSubnetGroup: configura las subredes donde se desplegará la base de datos. Por seguridad para esta arquitectura la base de datos se configurará en dos subredes privadas.

        ##Create the databases
        DBbook:
          Type: AWS::RDS::DBInstance
          Properties:
            AllocatedStorage: 20
            AvailabilityZone: us-east-1a
            DBInstanceIdentifier: databasebook
            DBInstanceClass: db.t3.micro
            DBName: dbbook
            DBSubnetGroupName: 
              Ref: DBbookSubnetGroup
            Engine: mariadb
            VPCSecurityGroups:
              - Ref: SGdb
            MasterUsername: root
            MasterUserPassword: '{{resolve:ssm-secure:/book/password:1}}'

        ##Create the subnets
        DBbookSubnetGroup:
          Type: AWS::RDS::DBSubnetGroup
          Properties:
            DBSubnetGroupDescription: Subnets about db
            DBSubnetGroupName: db-subnet-group-db
            SubnetIds:
              - Fn::ImportValue: !Sub "network-stack-PrivateSubnetAA"
              - Fn::ImportValue: !Sub "network-stack-PrivateSubnetBB"

- Con la configuración de **Outputs** en el servicio AWS Cloudformation en la pestaña de Output se puede ver el DNS generado por el Application Load Balancer. 

        Outputs:
          bookWSpublic:
            Description: Public IP of the book-ws
            Value: !GetAtt bookWSpublic.PublicIp
            Export:
              Name: !Sub "netwoRk-stack-book-ws-public"
          bookWSprivate:
            Description: Private IP of the book-ws
            Value: !GetAtt bookWSpublic.PrivateIp
            Export:
              Name: !Sub "network-stack-book-ws-private"
          bookWSALB:
            Description: ALB of the book-ws
            Value: !GetAtt ALBbook.DNSName
            Export:
              Name: !Sub "network-stack-book-ws-alb"
          SGdb:
            Description: SG for DB
            Value:
              Ref: SGdb
            Export:
              Name: !Sub "network-stack-SGdb"
          DBbook:
            Description: Database endpoint
            Value: !GetAtt DBbook.Endpoint.Address

<hr>

2. Después de escribir los templates, se hace el despliegue utilizando el AWS CLI.

- Primero se hace el despliegue de la infraestructura de red con el siguiente comando. 

        aws cloudformation create-stack --stack-name network-stack --template-body file://network.yml

- Después que el despliegue de la infraestructura de red sea exitoso, se hace el despliegue del stack de application.

      aws cloudformation create-stack --stack-name application-stack --template-body file://application.yml

3. Después que los despliegue de los stack sean exitosos, se hace la verificación de la implementación en AWS. 
  - Ir al servicio AWS Cloudformation
  - Seleccionar el Stack application-stack
  - Ir a la pestaña Output. 
  - Dar click en el DNS del Application Load Balancer. Se debe abrir en otra pestaña del navegador lo que muestra el servidor web. 
  - En la pestaña Libros se puede probar la conexión con la base de datos. 