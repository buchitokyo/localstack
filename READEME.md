## Localstack + SAM + Python
brew tap aws/tap  
brew install aws-sam-cli  
brew install awscl  

## AWS config
aws configure --profile=localstack  
AWS Access Key ID [None]: dummy  
AWS Secret Access Key [None]: dummy  
Default region name [None]: ap-northeast-1  
Default output format [None]: json  

## Make Python project
sam init --runtime python3.9 --name lambda  

├── READEME.md  
├── docker-compose.yml  
├── excel-auto  
│   ├── func  
│   │   ├── __init__.py  
│   │   ├── app.py  
│   │   └── requirements.txt  
│   └── template.yaml  
└── localstack  
    ├── cache  
    │   ├── machine.json  
    │   ├── server.test.pem  
    │   ├── server.test.pem.crt  
    │   └── server.test.pem.key  
    ├── data  
    │   └── startup_info.json  
    ├── logs  
    └── var_libs  

## template.yaml
remove API Gateway sections  
add Enviroment (S3)
```
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  excel-auto

  Sample SAM Template for excel-auto

# More info about Globals: https://github.com/awslabs/serverless-application-model/blob/master/docs/globals.rst
Globals:
  Function:
    Timeout: 3
    Tracing: Active
  Api:
    TracingEnabled: True

Resources:
  FuncFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: func/
      Handler: app.lambda_handler
      Runtime: python3.9
      Architectures:
        - x86_64
      # Events:
      #   HelloWorld:
      #     Type: Api # More info about API Event Source: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#api
      #     Properties:
      #       Path: /hello
      #       Method: get
      Environment:
        Variables:
          S3_BUCKET: 'develop'
          S3_PATH: '/'
          # SECRET_KEY: 'test_key'
          TZ: 'Asia/Tokyo'

Outputs:
  # ServerlessRestApi is an implicit API created out of Events key under Serverless::Function
  # Find out more about other implicit resources you can reference within SAM
  # https://github.com/awslabs/serverless-application-model/blob/master/docs/internals/generated_resources.rst#api
  # HelloWorldApi:
  #   Description: "API Gateway endpoint URL for Prod stage for Hello World function"
  #   Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod/hello/"
  FuncFunction:
    Description: "excel-auto Lambda Function ARN"
    Value: !GetAtt ExcelAutodFunction.Arn
  FuncFunctionIamRole:
    Description: "Implicit IAM Role created for excel-auto function"
    Value: !GetAtt ExcelAutoFunctionRole.Arn
```
remove event dir, readme.md, init.py .gitignore (if you have a repo, just leave i as it is) and etc..  

## docker
docker-compose up -d  
http://localhost:4566/health  

## S3
aws s3 mb s3://{bucket_name} --endpoint-url=http://localhost:4566 --profile=localstack  
aws s3 --endpoint-url=http://localhost:4566 cp ~/Downloads/{excel_name}.xlsx  s3://develop/excel/ --profile=localstack  
aws s3 ls --endpoint-url=http://localhost:4566 s3://develop/excel/  --profile=localstack  

## SAM build, invoke function
template.yamlがある場所で  
**`sam build --use-container`**

.aws-sam/build/template.yamlがある場所で  
**`sam local invoke FuncFunction --docker-network localstack.internal --log-file check.log --profile=localstack`**

問題なければ、  
**`sam deploy --guided`**
*AWSのクレデンシャルを設定、IAMロールの事前設定
