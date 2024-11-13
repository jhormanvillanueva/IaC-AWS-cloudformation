# Infraestructura como código utilizando AWS CloudFormation

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
