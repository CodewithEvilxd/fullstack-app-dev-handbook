# Lesson 44: Cloud Deployment & Scaling

## 🎯 **Learning Objectives**
- Master cloud deployment strategies for React Native applications
- Implement auto-scaling and load balancing
- Configure cloud databases and storage
- Set up monitoring and alerting in the cloud
- Optimize costs and performance in cloud environments

## 📚 **Table of Contents**
1. [Cloud Platforms Overview](#cloud-platforms-overview)
2. [AWS Deployment](#aws-deployment)
3. [Google Cloud Platform](#google-cloud-platform)
4. [Microsoft Azure](#microsoft-azure)
5. [Serverless Deployment](#serverless-deployment)
6. [Database as a Service](#database-as-a-service)
7. [Cloud Storage Solutions](#cloud-storage-solutions)
8. [CDN & Edge Computing](#cdn--edge-computing)
9. [Cost Optimization](#cost-optimization)
10. [Practical Examples](#practical-examples)

---

## ☁️ **Cloud Platforms Overview**

### **Major Cloud Providers**
- **AWS (Amazon Web Services)**: Most comprehensive cloud platform
- **Google Cloud Platform (GCP)**: Strong in AI/ML and data analytics
- **Microsoft Azure**: Enterprise-focused with strong hybrid capabilities
- **DigitalOcean**: Developer-friendly with simple pricing
- **Heroku**: Platform as a Service (PaaS) for easy deployments

### **Deployment Strategies**
- **Blue-Green Deployment**: Zero-downtime deployments with two environments
- **Canary Deployment**: Gradual rollout to subset of users
- **Rolling Deployment**: Update instances gradually
- **Immutable Deployment**: Replace entire infrastructure

### **Scaling Strategies**
- **Horizontal Scaling**: Add more instances
- **Vertical Scaling**: Increase instance size
- **Auto-scaling**: Automatic scaling based on metrics
- **Global Scaling**: Multi-region deployments

---

## 🏗️ **AWS Deployment**

### **AWS Services for React Native**
- **EC2**: Virtual servers for hosting
- **ECS/EKS**: Container orchestration
- **Lambda**: Serverless functions
- **API Gateway**: API management
- **CloudFront**: CDN for assets
- **S3**: Object storage
- **RDS**: Managed databases
- **ElastiCache**: In-memory caching
- **CloudWatch**: Monitoring and logging

### **EC2 Deployment**
```yaml
# cloudformation/ec2-deployment.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'React Native App Deployment on EC2'

Parameters:
  InstanceType:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
    Description: EC2 instance type

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Name of an existing EC2 KeyPair

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true

  Subnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs '']

  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for React Native app
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0

  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      ImageId: ami-0c55b159cbfafe1d0 # Amazon Linux 2
      SecurityGroupIds:
        - !Ref SecurityGroup
      SubnetId: !Ref Subnet
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          yum install -y docker
          systemctl start docker
          systemctl enable docker
          usermod -a -G docker ec2-user

          # Install Docker Compose
          curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
          chmod +x /usr/local/bin/docker-compose

          # Create app directory
          mkdir -p /app
          cd /app

Outputs:
  InstanceId:
    Description: Instance ID
    Value: !Ref EC2Instance
  PublicIP:
    Description: Public IP address
    Value: !GetAtt EC2Instance.PublicIp
```

### **ECS Deployment**
```json
// ecs-task-definition.json
{
  "family": "react-native-app",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "react-native-app",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/react-native-app:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8081,
          "hostPort": 8081,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "NODE_ENV",
          "value": "production"
        },
        {
          "name": "API_URL",
          "value": "https://api.myapp.com"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/react-native-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": [
          "CMD-SHELL",
          "curl -f http://localhost:8081/health || exit 1"
        ],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

### **API Gateway Setup**
```yaml
# api-gateway.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'API Gateway for React Native App'

Resources:
  ApiGateway:
    Type: AWS::ApiGateway::RestApi
    Properties:
      Name: ReactNativeAPI
      Description: API Gateway for React Native application
      EndpointConfiguration:
        Types:
          - REGIONAL

  ApiGatewayResource:
    Type: AWS::ApiGateway::Resource
    Properties:
      RestApiId: !Ref ApiGateway
      ParentId: !GetAtt ApiGateway.RootResourceId
      PathPart: 'api'

  ApiGatewayMethod:
    Type: AWS::ApiGateway::Method
    Properties:
      RestApiId: !Ref ApiGateway
      ResourceId: !Ref ApiGatewayResource
      HttpMethod: GET
      AuthorizationType: NONE
      Integration:
        Type: HTTP_PROXY
        IntegrationHttpMethod: GET
        Uri: !Sub 'http://${LoadBalancer.DNSName}/api'
        PassthroughBehavior: WHEN_NO_MATCH

  ApiGatewayDeployment:
    Type: AWS::ApiGateway::Deployment
    DependsOn: ApiGatewayMethod
    Properties:
      RestApiId: !Ref ApiGateway
      StageName: prod

  ApiGatewayStage:
    Type: AWS::ApiGateway::Stage
    Properties:
      RestApiId: !Ref ApiGateway
      DeploymentId: !Ref ApiGatewayDeployment
      StageName: prod
      MethodSettings:
        - ResourcePath: '/*'
          HttpMethod: '*'
          MetricsEnabled: true
          LoggingLevel: INFO
```

---

## ☁️ **Google Cloud Platform**

### **App Engine Deployment**
```yaml
# app.yaml
runtime: nodejs18
instance_class: F1

env_variables:
  NODE_ENV: production
  API_URL: https://api.myapp.com
  DATABASE_URL: postgresql://user:password@db:5432/myapp

handlers:
  - url: /api/.*
    script: auto
    secure: always

  - url: /.*
    static_files: build/index.html
    upload: build/index.html
    secure: always

  - url: /(.*)
    static_files: build/\1
    upload: build/(.*)
    secure: always

network:
  name: default
  subnetwork_name: default

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.7
  target_throughput_utilization: 0.7
```

### **Cloud Run Deployment**
```yaml
# cloud-run-service.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: react-native-api
  labels:
    cloud.googleapis.com/location: us-central1
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: '10'
        autoscaling.knative.dev/minScale: '1'
    spec:
      containers:
      - image: gcr.io/my-project/react-native-api:latest
        ports:
        - name: http1
          containerPort: 8081
        env:
        - name: NODE_ENV
          value: production
        - name: PORT
          value: '8081'
        resources:
          limits:
            cpu: 1000m
            memory: 512Mi
          requests:
            cpu: 500m
            memory: 256Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8081
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8081
          initialDelaySeconds: 5
          periodSeconds: 5
```

### **GKE Cluster Setup**
```yaml
# gke-cluster.yaml
apiVersion: container.googleapis.com/v1
kind: Cluster
metadata:
  name: react-native-cluster
  location: us-central1-a
spec:
  networking:
    services:
      networkingMode: VPC_NATIVE
      privateClusterConfig:
        enablePrivateNodes: true
        enablePrivateEndpoint: false
        masterIpv4CidrBlock: 172.16.0.0/28
    network: default
    subnetwork: default
  binaryAuthorization:
    evaluationMode: DISABLED
  clusterAutoscaling:
    enableNodeAutoprovisioning: true
    resourceLimits:
    - resourceType: cpu
      minimum: 1
      maximum: 8
    - resourceType: memory
      minimum: 2
      maximum: 16
  nodePools:
  - name: default-pool
    initialNodeCount: 3
    config:
      machineType: e2-medium
      diskSizeGb: 100
      oauthScopes:
      - https://www.googleapis.com/auth/cloud-platform
    management:
      autoRepair: true
      autoUpgrade: true
    autoscaling:
      enabled: true
      minNodeCount: 1
      maxNodeCount: 5
```

---

## 🔷 **Microsoft Azure**

### **Azure App Service**
```json
// azure-app-service.json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "appName": {
      "type": "string",
      "metadata": {
        "description": "Name of the app service"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Web/sites",
      "apiVersion": "2020-06-01",
      "name": "[parameters('appName')]",
      "location": "[parameters('location')]",
      "kind": "app,linux,container",
      "properties": {
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('appServicePlanName'))]",
        "siteConfig": {
          "linuxFxVersion": "DOCKER|myregistry.azurecr.io/react-native-app:latest",
          "appSettings": [
            {
              "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
              "value": "false"
            },
            {
              "name": "DOCKER_REGISTRY_SERVER_URL",
              "value": "https://myregistry.azurecr.io"
            },
            {
              "name": "NODE_ENV",
              "value": "production"
            }
          ],
          "alwaysOn": true,
          "healthCheckPath": "/health"
        }
      }
    },
    {
      "type": "Microsoft.Web/serverfarms",
      "apiVersion": "2020-06-01",
      "name": "[variables('appServicePlanName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "B1",
        "tier": "Basic",
        "size": "B1",
        "family": "B",
        "capacity": 1
      },
      "kind": "linux",
      "properties": {
        "perSiteScaling": false,
        "maximumElasticWorkerCount": 1,
        "isSpot": false,
        "targetWorkerCount": 0,
        "targetWorkerSizeId": 0
      }
    }
  ],
  "variables": {
    "appServicePlanName": "[concat(parameters('appName'), '-plan')]"
  }
}
```

### **Azure Kubernetes Service**
```yaml
# aks-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: react-native-deployment
  labels:
    app: react-native
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-native
  template:
    metadata:
      labels:
        app: react-native
    spec:
      containers:
      - name: react-native
        image: myregistry.azurecr.io/react-native-app:latest
        ports:
        - containerPort: 8081
        env:
        - name: NODE_ENV
          value: "production"
        - name: API_URL
          value: "https://api.myapp.com"
        resources:
          requests:
            cpu: 250m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        livenessProbe:
          httpGet:
            path: /health
            port: 8081
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8081
          initialDelaySeconds: 5
          periodSeconds: 5
      imagePullSecrets:
      - name: acr-secret

---
apiVersion: v1
kind: Service
metadata:
  name: react-native-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8081
  selector:
    app: react-native

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: react-native-ingress
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - host: myapp.azurewebsites.net
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: react-native-service
            port:
              number: 80
```

---

## ⚡ **Serverless Deployment**

### **AWS Lambda**
```javascript
// lambda/index.js
const express = require('express');
const serverless = require('serverless-http');

const app = express();

app.use(express.json());

// API routes
app.get('/api/health', (req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() });
});

app.get('/api/users', async (req, res) => {
  try {
    // Fetch users from database
    const users = await getUsersFromDB();
    res.json(users);
  } catch (error) {
    console.error('Error fetching users:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.post('/api/users', async (req, res) => {
  try {
    const userData = req.body;
    const newUser = await createUserInDB(userData);
    res.status(201).json(newUser);
  } catch (error) {
    console.error('Error creating user:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Database functions (simplified)
async function getUsersFromDB() {
  // Connect to database and fetch users
  return [
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com' },
  ];
}

async function createUserInDB(userData) {
  // Create user in database
  return {
    id: Date.now(),
    ...userData,
    createdAt: new Date().toISOString(),
  };
}

module.exports.handler = serverless(app);
```

```yaml
# serverless.yml
service: react-native-api

provider:
  name: aws
  runtime: nodejs18.x
  region: us-east-1
  stage: prod
  environment:
    NODE_ENV: production
    DATABASE_URL: ${env:DATABASE_URL}

  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:*
      Resource: "*"

functions:
  api:
    handler: index.handler
    events:
      - http:
          path: /
          method: ANY
          cors: true
      - http:
          path: /{proxy+}
          method: ANY
          cors: true

  health:
    handler: index.handler
    events:
      - http:
          path: /health
          method: GET

plugins:
  - serverless-offline
  - serverless-dotenv-plugin

custom:
  serverless-offline:
    httpPort: 3000
```

### **Google Cloud Functions**
```javascript
// functions/index.js
const functions = require('@google-cloud/functions-framework');
const express = require('express');

const app = express();

app.use(express.json());

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// API endpoints
app.get('/users', async (req, res) => {
  try {
    const users = await getUsers();
    res.json(users);
  } catch (error) {
    console.error('Error fetching users:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.post('/users', async (req, res) => {
  try {
    const userData = req.body;
    const newUser = await createUser(userData);
    res.status(201).json(newUser);
  } catch (error) {
    console.error('Error creating user:', error);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Database operations
async function getUsers() {
  // Fetch from Firestore or other database
  return [
    { id: 1, name: 'John Doe', email: 'john@example.com' },
  ];
}

async function createUser(userData) {
  // Create in Firestore or other database
  return {
    id: Date.now(),
    ...userData,
    createdAt: new Date().toISOString(),
  };
}

// Export for Google Cloud Functions
functions.http('api', app);
```

### **Vercel Deployment**
```javascript
// vercel/api/users.js
import { connectToDatabase } from '../../lib/mongodb';

export default async function handler(req, res) {
  const { db } = await connectToDatabase();

  if (req.method === 'GET') {
    try {
      const users = await db.collection('users').find({}).toArray();
      res.status(200).json(users);
    } catch (error) {
      console.error('Error fetching users:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  } else if (req.method === 'POST') {
    try {
      const userData = req.body;
      const result = await db.collection('users').insertOne({
        ...userData,
        createdAt: new Date(),
      });

      res.status(201).json({
        id: result.insertedId,
        ...userData,
      });
    } catch (error) {
      console.error('Error creating user:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  } else {
    res.setHeader('Allow', ['GET', 'POST']);
    res.status(405).json({ error: `Method ${req.method} not allowed` });
  }
}
```

```json
// vercel.json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/node"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "/api/$1"
    },
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ],
  "env": {
    "NODE_ENV": "production"
  }
}
```

---

## 🗄️ **Database as a Service**

### **AWS RDS**
```yaml
# rds-instance.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'RDS PostgreSQL instance for React Native app'

Parameters:
  DBInstanceClass:
    Type: String
    Default: db.t3.micro
    AllowedValues:
      - db.t3.micro
      - db.t3.small
      - db.t3.medium

  DBName:
    Type: String
    Default: myapp

  DBUsername:
    Type: String
    Default: myappuser

Resources:
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Subnet group for RDS instance
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2

  DBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: !Ref DBInstanceClass
      DBName: !Ref DBName
      Engine: postgres
      EngineVersion: '14.2'
      MasterUsername: !Ref DBUsername
      MasterUserPassword: !Ref DBPassword
      AllocatedStorage: '20'
      StorageType: gp2
      DBSubnetGroupName: !Ref DBSubnetGroup
      VPCSecurityGroups:
        - !Ref DBSecurityGroup
      BackupRetentionPeriod: 7
      MultiAZ: false
      PubliclyAccessible: false
      StorageEncrypted: true
      EnablePerformanceInsights: true
      PerformanceInsightsRetentionPeriod: 7

  DBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for RDS
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 5432
          ToPort: 5432
          SourceSecurityGroupId: !Ref AppSecurityGroup

Outputs:
  DBEndpoint:
    Description: Database endpoint
    Value: !GetAtt DBInstance.Endpoint.Address
  DBPort:
    Description: Database port
    Value: !GetAtt DBInstance.Endpoint.Port
```

### **Google Cloud SQL**
```yaml
# cloud-sql-instance.yaml
apiVersion: sql.cnrm.cloud.google.com/v1beta1
kind: SQLInstance
metadata:
  name: react-native-db
spec:
  region: us-central1
  databaseVersion: POSTGRES_14
  settings:
    tier: db-f1-micro
    diskSize: 10
    diskType: PD_SSD
    availabilityType: REGIONAL
    backupConfiguration:
      enabled: true
      startTime: "02:00"
    maintenanceWindow:
      day: 7
      hour: 2
    ipConfiguration:
      ipv4Enabled: false
      privateNetworkRef:
        name: default
    databaseFlags:
    - name: max_connections
      value: "100"
    - name: shared_preload_libraries
      value: "pg_stat_statements"

---
apiVersion: sql.cnrm.cloud.google.com/v1beta1
kind: SQLDatabase
metadata:
  name: react-native-database
spec:
  instanceRef:
    name: react-native-db

---
apiVersion: sql.cnrm.cloud.google.com/v1beta1
kind: SQLUser
metadata:
  name: react-native-user
spec:
  instanceRef:
    name: react-native-db
  password:
    value: "secure-password"
```

### **Azure Database**
```json
// azure-database.json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "serverName": {
      "type": "string",
      "metadata": {
        "description": "Name of the SQL server"
      }
    },
    "databaseName": {
      "type": "string",
      "defaultValue": "react-native-db",
      "metadata": {
        "description": "Name of the database"
      }
    },
    "administratorLogin": {
      "type": "string",
      "metadata": {
        "description": "Administrator login for the server"
      }
    },
    "administratorLoginPassword": {
      "type": "securestring",
      "metadata": {
        "description": "Administrator password for the server"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2020-11-01-preview",
      "name": "[parameters('serverName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "administratorLogin": "[parameters('administratorLogin')]",
        "administratorLoginPassword": "[parameters('administratorLoginPassword')]",
        "version": "12.0",
        "publicNetworkAccess": "Enabled",
        "minimalTlsVersion": "1.2"
      }
    },
    {
      "type": "Microsoft.Sql/servers/databases",
      "apiVersion": "2020-11-01-preview",
      "name": "[concat(parameters('serverName'), '/', parameters('databaseName'))]",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Basic",
        "tier": "Basic",
        "capacity": 5
      },
      "properties": {
        "collation": "SQL_Latin1_General_CP1_CI_AS",
        "maxSizeBytes": 1073741824,
        "zoneRedundant": false
      },
      "dependsOn": [
        "[resourceGroup().id, 'Microsoft.Sql/servers', parameters('serverName')]"
      ]
    },
    {
      "type": "Microsoft.Sql/servers/firewallRules",
      "apiVersion": "2020-11-01-preview",
      "name": "[concat(parameters('serverName'), '/AllowAllWindowsAzureIps')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "startIpAddress": "0.0.0.0",
        "endIpAddress": "0.0.0.0"
      },
      "dependsOn": [
        "[resourceGroup().id, 'Microsoft.Sql/servers', parameters('serverName')]"
      ]
    }
  ]
}
```

---

## ☁️ **Cloud Storage Solutions**

### **AWS S3**
```javascript
// services/s3Service.js
import AWS from 'aws-sdk';
import { RNS3 } from 'react-native-aws3';

class S3Service {
  constructor() {
    AWS.config.update({
      accessKeyId: process.env.AWS_ACCESS_KEY_ID,
      secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
      region: process.env.AWS_REGION,
    });

    this.s3 = new AWS.S3();
    this.bucketName = process.env.S3_BUCKET_NAME;
  }

  async uploadFile(fileUri, fileName, contentType = 'image/jpeg') {
    try {
      const file = {
        uri: fileUri,
        name: fileName,
        type: contentType,
      };

      const options = {
        keyPrefix: 'uploads/',
        bucket: this.bucketName,
        region: process.env.AWS_REGION,
        accessKey: process.env.AWS_ACCESS_KEY_ID,
        secretKey: process.env.AWS_SECRET_ACCESS_KEY,
        successActionStatus: 201,
      };

      const response = await RNS3.put(file, options);

      if (response.status === 201) {
        return {
          success: true,
          url: response.body.postResponse.location,
          key: response.body.postResponse.key,
        };
      } else {
        throw new Error('Upload failed');
      }
    } catch (error) {
      console.error('S3 upload error:', error);
      return {
        success: false,
        error: error.message,
      };
    }
  }

  async downloadFile(key) {
    try {
      const params = {
        Bucket: this.bucketName,
        Key: key,
      };

      const url = await this.s3.getSignedUrlPromise('getObject', params);
      return url;
    } catch (error) {
      console.error('S3 download error:', error);
      throw error;
    }
  }

  async deleteFile(key) {
    try {
      const params = {
        Bucket: this.bucketName,
        Key: key,
      };

      await this.s3.deleteObject(params).promise();
      return { success: true };
    } catch (error) {
      console.error('S3 delete error:', error);
      return {
        success: false,
        error: error.message,
      };
    }
  }

  async listFiles(prefix = '') {
    try {
      const params = {
        Bucket: this.bucketName,
        Prefix: prefix,
      };

      const response = await this.s3.listObjectsV2(params).promise();
      return response.Contents || [];
    } catch (error) {
      console.error('S3 list error:', error);
      throw error;
    }
  }

  getPublicUrl(key) {
    return `https://${this.bucketName}.s3.${process.env.AWS_REGION}.amazonaws.com/${key}`;
  }
}

export default new S3Service();
```

### **Google Cloud Storage**
```javascript
// services/gcsService.js
import { Storage } from '@google-cloud/storage';

class GCSService {
  constructor() {
    this.storage = new Storage({
      projectId: process.env.GCP_PROJECT_ID,
      keyFilename: process.env.GCP_KEY_FILE,
    });

    this.bucketName = process.env.GCS_BUCKET_NAME;
    this.bucket = this.storage.bucket(this.bucketName);
  }

  async uploadFile(filePath, destination, contentType = 'image/jpeg') {
    try {
      const options = {
        destination,
        metadata: {
          contentType,
        },
        public: true,
      };

      const [file] = await this.bucket.upload(filePath, options);

      return {
        success: true,
        url: `https://storage.googleapis.com/${this.bucketName}/${destination}`,
        name: file.name,
      };
    } catch (error) {
      console.error('GCS upload error:', error);
      return {
        success: false,
        error: error.message,
      };
    }
  }

  async downloadFile(fileName, destinationPath) {
    try {
      const file = this.bucket.file(fileName);
      await file.download({ destination: destinationPath });

      return {
        success: true,
        path: destinationPath,
      };
    } catch (error) {
      console.error('GCS download error:', error);
      return {
        success: false,
        error: error.message,
      };
    }
  }

  async deleteFile(fileName) {
    try {
      await this.bucket.file(fileName).delete();
      return { success: true };
    } catch (error) {
      console.error('GCS delete error:', error);
      return {
        success: false,
        error: error.message,
      };
    }
  }

  async listFiles(prefix = '') {
    try {
      const [files] = await this.bucket.getFiles({ prefix });
      return files.map(file => ({
        name: file.name,
        size: file.metadata.size,
        created: file.metadata.timeCreated,
        updated: file.metadata.updated,
      }));
    } catch (error) {
      console.error('GCS list error:', error);
      throw error;
    }
  }

  getPublicUrl(fileName) {
    return `https://storage.googleapis.com/${this.bucketName}/${fileName}`;
  }

  async generateSignedUrl(fileName, expiresInMinutes = 60) {
    try {
      const options = {
        version: 'v4',
        action: 'read',
        expires: Date.now() + expiresInMinutes * 60 * 1000,
      };

      const [url] = await this.bucket.file(fileName).getSignedUrl(options);
      return url;
    } catch (error) {
      console.error('GCS signed URL error:', error);
      throw error;
    }
  }
}

export default new GCSService();
```

---

## 🌐 **CDN & Edge Computing**

### **CloudFront Distribution**
```yaml
# cloudfront-distribution.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFront distribution for React Native app'

Parameters:
  S3BucketName:
    Type: String
    Description: Name of the S3 bucket

  PriceClass:
    Type: String
    Default: PriceClass_100
    AllowedValues:
      - PriceClass_100
      - PriceClass_200
      - PriceClass_All

Resources:
  CloudFrontOriginAccessIdentity:
    Type: AWS::CloudFront::OriginAccessIdentity
    Properties:
      OriginAccessIdentityConfig:
        Comment: !Sub 'OAI for ${S3BucketName}'

  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Comment: React Native app distribution
        DefaultCacheBehavior:
          TargetOriginId: !Sub 'S3-${S3BucketName}'
          ViewerProtocolPolicy: redirect-to-https
          Compress: true
          ForwardedValues:
            QueryString: false
            Cookies:
              Forward: none
          CachePolicyId: '4135ea2d-6df8-44a3-9df3-4b5a84be39ad' # CachingEnabled
        Origins:
        - DomainName: !Sub '${S3BucketName}.s3.amazonaws.com'
          Id: !Sub 'S3-${S3BucketName}'
          S3OriginConfig:
            OriginAccessIdentity: !Sub 'origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}'
        Enabled: true
        DefaultRootObject: index.html
        PriceClass: !Ref PriceClass
        HttpVersion: http2
        CacheBehaviors:
        - PathPattern: 'static/*'
          TargetOriginId: !Sub 'S3-${S3BucketName}'
          ViewerProtocolPolicy: redirect-to-https
          Compress: true
          ForwardedValues:
            QueryString: false
            Cookies:
              Forward: none
          CachePolicyId: '4135ea2d-6df8-44a3-9df3-4b5a84be39ad'
        CustomErrorResponses:
        - ErrorCode: 404
          ResponsePagePath: /index.html
          ResponseCode: 200
          ErrorCachingMinTTL: 300

Outputs:
  DistributionId:
    Description: CloudFront distribution ID
    Value: !Ref CloudFrontDistribution
  DistributionURL:
    Description: CloudFront distribution URL
    Value: !Sub 'https://${CloudFrontDistribution.DomainName}'
```

### **Cloudflare Workers**
```javascript
// cloudflare-worker.js
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const url = new URL(request.url)

  // API routing
  if (url.pathname.startsWith('/api/')) {
    return handleAPIRequest(request)
  }

  // Static asset caching
  if (url.pathname.match(/\.(css|js|png|jpg|jpeg|gif|ico|svg)$/)) {
    return handleStaticAsset(request)
  }

  // Default response
  return fetch(request)
}

async function handleAPIRequest(request) {
  const url = new URL(request.url)

  // Add CORS headers
  const corsHeaders = {
    'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type, Authorization',
  }

  // Handle preflight requests
  if (request.method === 'OPTIONS') {
    return new Response(null, { headers: corsHeaders })
  }

  // Route to appropriate backend
  let backendURL

  if (url.pathname.startsWith('/api/users')) {
    backendURL = 'https://user-service.myapp.com'
  } else if (url.pathname.startsWith('/api/products')) {
    backendURL = 'https://product-service.myapp.com'
  } else {
    backendURL = 'https://api.myapp.com'
  }

  // Proxy the request
  const response = await fetch(`${backendURL}${url.pathname}${url.search}`, {
    method: request.method,
    headers: request.headers,
    body: request.body,
  })

  // Add CORS headers to response
  const newResponse = new Response(response.body, response)
  Object.keys(corsHeaders).forEach(key => {
    newResponse.headers.set(key, corsHeaders[key])
  })

  return newResponse
}

async function handleStaticAsset(request) {
  // Check cache first
  const cache = caches.default
  let response = await cache.match(request)

  if (!response) {
    // Fetch from origin
    response = await fetch(request)

    // Cache the response
    const cacheResponse = response.clone()
    cache.put(request, cacheResponse)
  }

  return response
}
```

---

## 💰 **Cost Optimization**

### **AWS Cost Optimization**
```javascript
// services/costOptimization.js
import AWS from 'aws-sdk';

class AWSCostOptimizer {
  constructor() {
    this.cloudwatch = new AWS.CloudWatch();
    this.ec2 = new AWS.EC2();
    this.rds = new AWS.RDS();
  }

  async getEC2Utilization() {
    const params = {
      Namespace: 'AWS/EC2',
      MetricName: 'CPUUtilization',
      Dimensions: [
        {
          Name: 'InstanceId',
          Value: 'i-1234567890abcdef0', // Replace with actual instance ID
        },
      ],
      StartTime: new Date(Date.now() - 24 * 60 * 60 * 1000), // 24 hours ago
      EndTime: new Date(),
      Period: 3600, // 1 hour
      Statistics: ['Average'],
    };

    try {
      const data = await this.cloudwatch.getMetricStatistics(params).promise();
      return data.Datapoints;
    } catch (error) {
      console.error('Error getting EC2 utilization:', error);
      return [];
    }
  }

  async getRDSUtilization() {
    const params = {
      DBInstanceIdentifier: 'my-db-instance',
      StartTime: new Date(Date.now() - 24 * 60 * 60 * 1000),
      EndTime: new Date(),
      MetricName: 'CPUUtilization',
      Period: 3600,
    };

    try {
      const data = await this.rds.getMetricStatistics(params).promise();
      return data.Datapoints;
    } catch (error) {
      console.error('Error getting RDS utilization:', error);
      return [];
    }
  }

  async suggestInstanceType(currentInstanceType, utilization) {
    const averageUtilization = utilization.reduce((sum, point) => sum + point.Average, 0) / utilization.length;

    if (averageUtilization < 20) {
      return {
        suggestion: 'downsize',
        recommendedType: this.getSmallerInstanceType(currentInstanceType),
        potentialSavings: this.calculateSavings(currentInstanceType, this.getSmallerInstanceType(currentInstanceType)),
      };
    } else if (averageUtilization > 80) {
      return {
        suggestion: 'upsize',
        recommendedType: this.getLargerInstanceType(currentInstanceType),
        potentialCost: this.calculateCostIncrease(currentInstanceType, this.getLargerInstanceType(currentInstanceType)),
      };
    }

    return {
      suggestion: 'optimal',
      message: 'Current instance type is appropriate for the workload',
    };
  }

  getSmallerInstanceType(instanceType) {
    const instanceTypes = ['t3.micro', 't3.small', 't3.medium', 't3.large', 't3.xlarge'];
    const currentIndex = instanceTypes.indexOf(instanceType);

    if (currentIndex > 0) {
      return instanceTypes[currentIndex - 1];
    }

    return instanceType;
  }

  getLargerInstanceType(instanceType) {
    const instanceTypes = ['t3.micro', 't3.small', 't3.medium', 't3.large', 't3.xlarge'];
    const currentIndex = instanceTypes.indexOf(instanceType);

    if (currentIndex < instanceTypes.length - 1) {
      return instanceTypes[currentIndex + 1];
    }

    return instanceType;
  }

  calculateSavings(currentType, recommendedType) {
    // Simplified cost calculation (actual costs vary by region)
    const costs = {
      't3.micro': 0.0104,
      't3.small': 0.0208,
      't3.medium': 0.0416,
      't3.large': 0.0832,
      't3.xlarge': 0.1664,
    };

    const currentCost = costs[currentType] || 0;
    const recommendedCost = costs[recommendedType] || 0;
    const monthlySavings = (currentCost - recommendedCost) * 24 * 30;

    return monthlySavings;
  }

  calculateCostIncrease(currentType, recommendedType) {
    const costs = {
      't3.micro': 0.0104,
      't3.small': 0.0208,
      't3.medium': 0.0416,
      't3.large': 0.0832,
      't3.xlarge': 0.1664,
    };

    const currentCost = costs[currentType] || 0;
    const recommendedCost = costs[recommendedType] || 0;
    const monthlyIncrease = (recommendedCost - currentCost) * 24 * 30;

    return monthlyIncrease;
  }

  async generateCostReport() {
    const [ec2Utilization, rdsUtilization] = await Promise.all([
      this.getEC2Utilization(),
      this.getRDSUtilization(),
    ]);

    const ec2Suggestion = await this.suggestInstanceType('t3.medium', ec2Utilization);

    return {
      ec2: {
        utilization: ec2Utilization,
        suggestion: ec2Suggestion,
      },
      rds: {
        utilization: rdsUtilization,
      },
      totalPotentialSavings: ec2Suggestion.potentialSavings || 0,
      generatedAt: new Date().toISOString(),
    };
  }
}

export default AWSCostOptimizer;
```

### **Auto Scaling Configuration**
```yaml
# auto-scaling-group.yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Auto Scaling Group for React Native app'

Resources:
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: react-native-launch-template
      LaunchTemplateData:
        ImageId: ami-0c55b159cbfafe1d0
        InstanceType: t3.micro
        KeyName: my-key-pair
        SecurityGroupIds:
          - !Ref InstanceSecurityGroup
        UserData:
          Fn::Base64: |
            #!/bin/bash
            yum update -y
            # Install application dependencies

  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      AutoScalingGroupName: react-native-asg
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: '1'
      MinSize: '1'
      MaxSize: '10'
      DesiredCapacity: '2'
      AvailabilityZones:
        - !Select [0, !GetAZs '']
      HealthCheckType: EC2
      HealthCheckGracePeriod: 300
      TargetGroupARNs:
        - !Ref TargetGroup

  ScaleOutPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: 70.0

  ScaleInPolicy:
    Type: AWS::AutoScaling::ScalingPolicy
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      PolicyType: TargetTrackingScaling
      TargetTrackingConfiguration:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: 30.0

  # Scheduled scaling for predictable traffic
  ScheduledScaleOut:
    Type: AWS::AutoScaling::ScheduledAction
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      ScheduledActionName: scale-out-business-hours
      Recurrence: "0 9 * * MON-FRI"  # 9 AM weekdays
      MinSize: "3"
      MaxSize: "8"
      DesiredCapacity: "5"

  ScheduledScaleIn:
    Type: AWS::AutoScaling::ScheduledAction
    Properties:
      AutoScalingGroupName: !Ref AutoScalingGroup
      ScheduledActionName: scale-in-off-hours
      Recurrence: "0 18 * * MON-FRI"  # 6 PM weekdays
      MinSize: "1"
      MaxSize: "3"
      DesiredCapacity: "2"

---

## 🎯 **Practical Examples**

### **Complete AWS Deployment Pipeline**
```yaml
# .github/workflows/aws-deploy.yml
name: Deploy to AWS

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
    - name: Install dependencies
      run: npm ci
    - name: Run tests
      run: npm test
    - name: Build application
      run: npm run build

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: staging
    steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build and push Docker image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: react-native-app
        IMAGE_TAG: staging-${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

    - name: Deploy to ECS
      run: |
        aws ecs update-service \
          --cluster react-native-staging \
          --service react-native-service \
          --force-new-deployment \
          --task-definition react-native-staging

  deploy-production:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: Build and push Docker image
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: react-native-app
        IMAGE_TAG: prod-${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

    - name: Run database migrations
      run: |
        aws ecs run-task \
          --cluster react-native-prod \
          --task-definition migration-task \
          --overrides '{
            "containerOverrides": [{
              "name": "migration-container",
              "environment": [
                {"name": "NODE_ENV", "value": "production"},
                {"name": "DB_MIGRATION", "value": "true"}
              ]
            }]
          }'

    - name: Deploy to ECS with blue-green
      run: |
        # Get current task definition
        TASK_DEF=$(aws ecs describe-task-definition --task-definition react-native-prod --query 'taskDefinition.taskDefinitionArn' --output text)

        # Update service with new task definition
        aws ecs update-service \
          --cluster react-native-prod \
          --service react-native-service \
          --task-definition $TASK_DEF \
          --force-new-deployment

    - name: Health check
      run: |
        # Wait for deployment to complete
        aws ecs wait services-stable \
          --cluster react-native-prod \
          --services react-native-service

        # Run health checks
        HEALTH_CHECK_URL="https://api.myapp.com/health"
        for i in {1..30}; do
          if curl -f $HEALTH_CHECK_URL; then
            echo "Health check passed"
            break
          fi
          echo "Waiting for health check..."
          sleep 10
        done

    - name: Rollback on failure
      if: failure()
      run: |
        echo "Deployment failed, initiating rollback..."
        aws ecs update-service \
          --cluster react-native-prod \
          --service react-native-service \
          --task-definition react-native-prod-previous \
          --force-new-deployment
```

### **Multi-Region Deployment Strategy**
```javascript
// services/globalDeployment.js
class GlobalDeploymentManager {
  constructor() {
    this.regions = {
      primary: 'us-east-1',
      secondary: ['us-west-2', 'eu-west-1', 'ap-southeast-1'],
    };

    this.services = {
      ec2: null,
      route53: null,
      cloudfront: null,
    };
  }

  async deployToRegion(region, config) {
    console.log(`Deploying to region: ${region}`);

    // Create VPC and networking
    const vpcId = await this.createVPC(region, config);

    // Deploy application infrastructure
    const infrastructure = await this.deployInfrastructure(region, vpcId, config);

    // Configure load balancer
    const loadBalancer = await this.configureLoadBalancer(region, infrastructure);

    // Update DNS
    await this.updateDNS(region, loadBalancer);

    return {
      region,
      vpcId,
      infrastructure,
      loadBalancer,
      deployedAt: new Date().toISOString(),
    };
  }

  async createVPC(region, config) {
    const ec2 = new AWS.EC2({ region });

    const vpcParams = {
      CidrBlock: config.vpcCidr || '10.0.0.0/16',
    };

    const vpc = await ec2.createVpc(vpcParams).promise();

    // Create subnets
    const subnetParams = {
      VpcId: vpc.Vpc.VpcId,
      CidrBlock: '10.0.1.0/24',
      AvailabilityZone: `${region}a`,
    };

    const subnet = await ec2.createSubnet(subnetParams).promise();

    return vpc.Vpc.VpcId;
  }

  async deployInfrastructure(region, vpcId, config) {
    // Deploy ECS cluster
    const clusterName = `react-native-${region}`;
    const ecs = new AWS.ECS({ region });

    await ecs.createCluster({ clusterName }).promise();

    // Deploy RDS instance
    const rds = new AWS.RDS({ region });
    const dbParams = {
      DBInstanceIdentifier: `react-native-db-${region}`,
      DBInstanceClass: 'db.t3.micro',
      Engine: 'postgres',
      MasterUsername: 'admin',
      MasterUserPassword: config.dbPassword,
      AllocatedStorage: 20,
      DBName: 'reactnative',
    };

    const dbInstance = await rds.createDBInstance(dbParams).promise();

    return {
      clusterName,
      dbInstance: dbInstance.DBInstance.DBInstanceIdentifier,
    };
  }

  async configureLoadBalancer(region, infrastructure) {
    const elbv2 = new AWS.ELBv2({ region });

    const lbParams = {
      Name: `react-native-lb-${region}`,
      Subnets: [], // Add subnet IDs
      SecurityGroups: [], // Add security group IDs
      Scheme: 'internet-facing',
      Type: 'application',
      IpAddressType: 'ipv4',
    };

    const loadBalancer = await elbv2.createLoadBalancer(lbParams).promise();

    // Create target group
    const tgParams = {
      Name: `react-native-tg-${region}`,
      Protocol: 'HTTP',
      Port: 80,
      VpcId: '', // Add VPC ID
      TargetType: 'ip',
    };

    const targetGroup = await elbv2.createTargetGroup(tgParams).promise();

    return {
      loadBalancerArn: loadBalancer.LoadBalancers[0].LoadBalancerArn,
      targetGroupArn: targetGroup.TargetGroups[0].TargetGroupArn,
      dnsName: loadBalancer.LoadBalancers[0].DNSName,
    };
  }

  async updateDNS(region, loadBalancer) {
    const route53 = new AWS.Route53();

    // Get hosted zone
    const zones = await route53.listHostedZones().promise();
    const hostedZone = zones.HostedZones.find(zone =>
      zone.Name.includes('myapp.com')
    );

    if (!hostedZone) {
      throw new Error('Hosted zone not found');
    }

    // Create or update record
    const recordParams = {
      HostedZoneId: hostedZone.Id,
      ChangeBatch: {
        Changes: [{
          Action: 'UPSERT',
          ResourceRecordSet: {
            Name: `${region}.api.myapp.com`,
            Type: 'CNAME',
            TTL: 300,
            ResourceRecords: [{
              Value: loadBalancer.dnsName,
            }],
          },
        }],
      },
    };

    await route53.changeResourceRecordSets(recordParams).promise();
  }

  async deployGlobally(config) {
    const deployments = [];

    // Deploy to primary region first
    console.log('Starting global deployment...');
    const primaryDeployment = await this.deployToRegion(this.regions.primary, config);
    deployments.push(primaryDeployment);

    // Deploy to secondary regions
    for (const region of this.regions.secondary) {
      try {
        const deployment = await this.deployToRegion(region, config);
        deployments.push(deployment);
      } catch (error) {
        console.error(`Failed to deploy to ${region}:`, error);
        // Continue with other regions
      }
    }

    // Configure global load balancing
    await this.configureGlobalLoadBalancing(deployments);

    return deployments;
  }

  async configureGlobalLoadBalancing(deployments) {
    const route53 = new AWS.Route53();

    // Create latency-based routing
    const zones = await route53.listHostedZones().promise();
    const hostedZone = zones.HostedZones.find(zone =>
      zone.Name.includes('myapp.com')
    );

    const changes = deployments.map(deployment => ({
      Action: 'CREATE',
      ResourceRecordSet: {
        Name: 'api.myapp.com',
        Type: 'CNAME',
        SetIdentifier: deployment.region,
        Weight: 100,
        Region: deployment.region,
        TTL: 60,
        ResourceRecords: [{
          Value: deployment.loadBalancer.dnsName,
        }],
      },
    }));

    const recordParams = {
      HostedZoneId: hostedZone.Id,
      ChangeBatch: {
        Changes: changes,
      },
    };

    await route53.changeResourceRecordSets(recordParams).promise();
  }

  async getDeploymentStatus() {
    const statuses = [];

    for (const region of [this.regions.primary, ...this.regions.secondary]) {
      try {
        const ecs = new AWS.ECS({ region });
        const clusters = await ecs.listClusters().promise();

        const cluster = clusters.clusterArns.find(arn =>
          arn.includes('react-native')
        );

        if (cluster) {
          const services = await ecs.describeServices({
            cluster,
            services: ['react-native-service'],
          }).promise();

          statuses.push({
            region,
            status: 'deployed',
            cluster,
            serviceCount: services.services.length,
          });
        } else {
          statuses.push({
            region,
            status: 'not_deployed',
          });
        }
      } catch (error) {
        statuses.push({
          region,
          status: 'error',
          error: error.message,
        });
      }
    }

    return statuses;
  }
}

export default GlobalDeploymentManager;
```

### **Cloud Monitoring Dashboard**
```javascript
// services/monitoringDashboard.js
import AWS from 'aws-sdk';
import { CloudWatch, CostExplorer } from 'aws-sdk';

class CloudMonitoringDashboard {
  constructor() {
    this.cloudwatch = new CloudWatch();
    this.costExplorer = new CostExplorer();
    this.metrics = [];
  }

  async getSystemMetrics(timeRange = 24) {
    const endTime = new Date();
    const startTime = new Date(endTime.getTime() - timeRange * 60 * 60 * 1000);

    const metrics = [
      {
        name: 'CPUUtilization',
        namespace: 'AWS/EC2',
        dimensions: [{ Name: 'InstanceId', Value: 'i-1234567890abcdef0' }],
      },
      {
        name: 'DatabaseConnections',
        namespace: 'AWS/RDS',
        dimensions: [{ Name: 'DBInstanceIdentifier', Value: 'my-db-instance' }],
      },
      {
        name: 'RequestCount',
        namespace: 'AWS/ApiGateway',
        dimensions: [{ Name: 'ApiName', Value: 'ReactNativeAPI' }],
      },
    ];

    const results = {};

    for (const metric of metrics) {
      const params = {
        Namespace: metric.namespace,
        MetricName: metric.name,
        Dimensions: metric.dimensions,
        StartTime: startTime,
        EndTime: endTime,
        Period: 300, // 5 minutes
        Statistics: ['Average', 'Maximum', 'Minimum'],
      };

      try {
        const data = await this.cloudwatch.getMetricStatistics(params).promise();
        results[metric.name] = {
          datapoints: data.Datapoints,
          unit: data.Datapoints[0]?.Unit || 'Count',
        };
      } catch (error) {
        console.error(`Error fetching ${metric.name}:`, error);
        results[metric.name] = { error: error.message };
      }
    }

    return results;
  }

  async getCostAnalysis(timeRange = 30) {
    const endDate = new Date();
    const startDate = new Date(endDate.getTime() - timeRange * 24 * 60 * 60 * 1000);

    const params = {
      TimePeriod: {
        Start: startDate.toISOString().split('T')[0],
        End: endDate.toISOString().split('T')[0],
      },
      Granularity: 'DAILY',
      Metrics: ['BlendedCost'],
      GroupBy: [
        {
          Type: 'DIMENSION',
          Key: 'SERVICE',
        },
      ],
    };

    try {
      const data = await this.costExplorer.getCostAndUsage(params).promise();

      const costs = {};
      data.ResultsByTime.forEach(result => {
        result.Groups.forEach(group => {
          const service = group.Keys[0];
          const amount = parseFloat(group.Metrics.BlendedCost.Amount);

          if (!costs[service]) {
            costs[service] = [];
          }

          costs[service].push({
            date: result.TimePeriod.Start,
            amount,
            unit: group.Metrics.BlendedCost.Unit,
          });
        });
      });

      return costs;
    } catch (error) {
      console.error('Error fetching cost data:', error);
      throw error;
    }
  }

  async getPerformanceInsights() {
    const insights = {
      bottlenecks: [],
      recommendations: [],
      alerts: [],
    };

    // Check for high CPU usage
    const cpuMetrics = await this.getSystemMetrics(1); // Last hour
    const avgCpu = cpuMetrics.CPUUtilization?.datapoints?.reduce(
      (sum, point) => sum + point.Average, 0
    ) / cpuMetrics.CPUUtilization?.datapoints?.length;

    if (avgCpu > 80) {
      insights.bottlenecks.push({
        type: 'high_cpu',
        severity: 'high',
        message: `Average CPU utilization is ${avgCpu.toFixed(1)}%`,
        recommendation: 'Consider scaling up instance or optimizing application',
      });
    }

    // Check database connections
    const dbConnections = cpuMetrics.DatabaseConnections?.datapoints?.reduce(
      (sum, point) => sum + point.Average, 0
    ) / cpuMetrics.DatabaseConnections?.datapoints?.length;

    if (dbConnections > 100) {
      insights.bottlenecks.push({
        type: 'high_db_connections',
        severity: 'medium',
        message: `Average database connections: ${dbConnections.toFixed(0)}`,
        recommendation: 'Consider connection pooling or database optimization',
      });
    }

    // Cost optimization suggestions
    const costs = await this.getCostAnalysis(7); // Last week
    const totalCost = Object.values(costs).reduce((sum, serviceCosts) =>
      sum + serviceCosts.reduce((serviceSum, cost) => serviceSum + cost.amount, 0), 0
    );

    if (totalCost > 100) { // Arbitrary threshold
      insights.recommendations.push({
        type: 'cost_optimization',
        message: `Weekly cost: $${totalCost.toFixed(2)}`,
        suggestions: [
          'Review unused resources',
          'Consider reserved instances',
          'Optimize storage usage',
        ],
      });
    }

    return insights;
  }

  async generateReport() {
    const [metrics, costs, insights] = await Promise.all([
      this.getSystemMetrics(24), // Last 24 hours
      this.getCostAnalysis(30), // Last 30 days
      this.getPerformanceInsights(),
    ]);

    return {
      generatedAt: new Date().toISOString(),
      timeRange: {
        metrics: '24 hours',
        costs: '30 days',
      },
      metrics,
      costs,
      insights,
      summary: {
        totalServices: Object.keys(costs).length,
        totalCost: Object.values(costs).reduce((sum, serviceCosts) =>
          sum + serviceCosts.reduce((serviceSum, cost) => serviceSum + cost.amount, 0), 0
        ),
        alertsCount: insights.alerts.length,
        recommendationsCount: insights.recommendations.length,
      },
    };
  }

  async setupAlerts() {
    const alarms = [
      {
        AlarmName: 'HighCPUUtilization',
        ComparisonOperator: 'GreaterThanThreshold',
        EvaluationPeriods: 2,
        MetricName: 'CPUUtilization',
        Namespace: 'AWS/EC2',
        Period: 300,
        Statistic: 'Average',
        Threshold: 80.0,
        ActionsEnabled: true,
        AlarmActions: [], // Add SNS topic ARN
        Dimensions: [
          {
            Name: 'InstanceId',
            Value: 'i-1234567890abcdef0',
          },
        ],
      },
      {
        AlarmName: 'HighDatabaseConnections',
        ComparisonOperator: 'GreaterThanThreshold',
        EvaluationPeriods: 2,
        MetricName: 'DatabaseConnections',
        Namespace: 'AWS/RDS',
        Period: 300,
        Statistic: 'Average',
        Threshold: 50,
        ActionsEnabled: true,
        AlarmActions: [], // Add SNS topic ARN
        Dimensions: [
          {
            Name: 'DBInstanceIdentifier',
            Value: 'my-db-instance',
          },
        ],
      },
    ];

    for (const alarm of alarms) {
      try {
        await this.cloudwatch.putMetricAlarm(alarm).promise();
        console.log(`Created alarm: ${alarm.AlarmName}`);
      } catch (error) {
        console.error(`Error creating alarm ${alarm.AlarmName}:`, error);
      }
    }
  }
}

export default CloudMonitoringDashboard;
```

---

## 📝 **Lesson Summary**

### **Key Concepts Learned**
- ✅ **Cloud Platforms**: AWS, Google Cloud, Azure deployment strategies
- ✅ **Infrastructure as Code**: CloudFormation, Terraform, Kubernetes manifests
- ✅ **Container Orchestration**: Docker, ECS, Kubernetes deployments
- ✅ **Serverless Computing**: Lambda, Cloud Functions, API Gateway
- ✅ **Database Services**: RDS, Cloud SQL, Azure Database
- ✅ **Cloud Storage**: S3, Cloud Storage, Blob Storage
- ✅ **CDN & Edge Computing**: CloudFront, Cloudflare Workers
- ✅ **Auto-scaling**: EC2 Auto Scaling, Kubernetes HPA
- ✅ **Cost Optimization**: Resource monitoring, right-sizing, reserved instances
- ✅ **Monitoring & Alerting**: CloudWatch, Stackdriver, Application Insights

### **Best Practices**
1. **Infrastructure as Code**: Use declarative configurations for reproducible deployments
2. **Multi-region Deployment**: Distribute applications across regions for high availability
3. **Auto-scaling**: Implement automatic scaling based on metrics and schedules
4. **Cost Monitoring**: Regularly review and optimize cloud spending
5. **Security First**: Implement least privilege, encryption, and compliance
6. **Monitoring**: Set up comprehensive monitoring and alerting
7. **Backup Strategy**: Implement automated backups and disaster recovery
8. **Performance Testing**: Load test applications before production deployment
9. **CI/CD Pipeline**: Automate testing, building, and deployment processes
10. **Documentation**: Maintain up-to-date infrastructure and deployment documentation

### **Next Steps**
- Learn about Infrastructure as Code with Terraform and Pulumi
- Study Kubernetes advanced patterns (operators, custom resources)
- Explore serverless architectures and event-driven systems
- Learn about cloud security (IAM, VPC, encryption)
- Study DevOps practices and site reliability engineering
- Explore multi-cloud and hybrid cloud architectures
- Learn about chaos engineering and resilience testing
- Study cloud cost management and optimization tools
- Explore edge computing and IoT integration
- Learn about cloud-native application development

---

## 🎯 **Assignment**

### **Task 1: Complete AWS Deployment**
Create a complete AWS deployment for a React Native application with:
- Infrastructure as Code using CloudFormation
- ECS cluster with Fargate for containerized deployment
- RDS PostgreSQL database with multi-AZ setup
- S3 bucket for file storage with CloudFront CDN
- API Gateway with proper authentication and rate limiting
- Auto-scaling based on CPU and memory utilization
- CloudWatch monitoring and alerting
- CI/CD pipeline with GitHub Actions
- Cost optimization and budget alerts

### **Task 2: Multi-Cloud Deployment**
Implement a multi-cloud deployment strategy with:
- Primary deployment on AWS with backup on Google Cloud
- Global load balancing with Route 53 and Cloud Load Balancing
- Database replication across cloud providers
- Automated failover and disaster recovery
- Cross-cloud monitoring and alerting
- Cost comparison and optimization across providers
- Data synchronization and consistency
- Backup and recovery across cloud providers

### **Task 3: Serverless Architecture**
Build a complete serverless architecture with:
- API Gateway with Lambda functions for backend logic
- DynamoDB for NoSQL data storage
- S3 for file storage and static website hosting
- CloudFront for global CDN distribution
- Cognito for user authentication and authorization
- Step Functions for complex workflows
- EventBridge for event-driven processing
- CloudWatch for monitoring and logging
- Cost optimization with provisioned concurrency

### **Task 4: Kubernetes Production Setup**
Create a production-ready Kubernetes deployment with:
- Multi-node cluster setup with high availability
- Ingress controller with SSL termination
- Helm charts for application deployment
- ConfigMaps and Secrets management
- Persistent volumes for database storage
- Network policies for security
- Horizontal Pod Autoscaler for automatic scaling
- Prometheus and Grafana for monitoring
- CI/CD pipeline with automated deployments
- Backup and disaster recovery procedures

---

## 📚 **Additional Resources**
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Google Cloud Architecture Center](https://cloud.google.com/architecture)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Terraform Documentation](https://www.terraform.io/docs/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [Cloud Cost Optimization](https://aws.amazon.com/aws-cost-management/)

---

**Next Lesson**: [Lesson 45: Security Best Practices](Lesson%2045_%20Security%20Best%20Practices.md)
