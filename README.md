# Infraestructura como código utilizando AWS CloudFormation
![AWS](https://img.shields.io/badge/Amazon_AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)

En este proyecto vamos a lanzar una infraestructura en la nube que sea altamente disponible, escalable y segura usando las buenas prácticas del well-architect framework. 

El despliegue de la arquitectura se hará por capas utilizando el servicio Cloudformation que es un servicio de AWS de infraestructura como código. La arquitectura a desplegar es la que se muestra en la siguiente imagen. 

![arquitectura](img/arquitecturaHA-HS.png)


<hr>

1. Vamos a crear primero los templates usando la extensión .yml

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
  - Routes: se definen las rutas que se asocial a las tablas de enrutamiento. 


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

2. Vamos a lanzar el template usando el AWS CLI

        aws cloudformation create-stack --stack-name network-stack --template-body file://network.yml

## Vamos a verificar la implementación del stack en AWS