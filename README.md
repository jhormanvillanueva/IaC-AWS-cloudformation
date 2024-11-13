# Infraestructura como código utilizando AWS CloudFormation
![AWS](https://img.shields.io/badge/Amazon_AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)

En este proyecto vamos a lanzar una infraestructura en la nube usando las buenas prácticas del well-architect framework

![arquitectura](img/arquitecturaHA-HS.png)


<hr>

1. Vamos a crear primero los templates usando la extensión .yml

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

2. Vamos a lanzar el template usando el AWS CLI

        aws cloudformation create-stack --stack-name network-stack --template-body file://network.yml

## Vamos a verificar la implementación del stack en AWS