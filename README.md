

```
az login --use-device-code

## 下記を実行でAzureのサブスクリプションIDを取得
az account list \
  --refresh \
  --query "[].id" \
  --output table

## <your subscription id>の部分を、上記で取得したAzureのサブスクリプションIDに置き換える
az account set --subscription <your subscription id>

group_name=20220615LineBot

az group create --name ${group_name} --location japaneast

az configure --defaults group=${group_name}

containerVer=v1
gitRepository=＜your ripogitry＞

az deployment group create --name deployAcr01 --template-file containerRegistry.bicep \
 --parameters containerVer=${containerVer} \
 --parameters gitRepository=${gitRepository}

# ACRの名前取得
az deployment group show \
 -g ${group_name} \
 -n deployAcr01 \
 --query properties.outputs.acrName.value

containerRegistryName=<上記で取得したACRの名前>
secret=<your secret>
access=<your access token>
## ココでrandomな値を取得 > https://1password.com/jp/password-generator/
random=<random value>
appsPort=5000

## secretとaccessはLINE Developersのチャネル設定でメモした値を入れる
az deployment group create --name deployPrj01 --template-file main.bicep \
 --parameters containerVer=${containerVer} \
 --parameters containerRegistryName=${containerRegistryName} \
 --parameters random=${random} \
 --parameters secret=${secret} \
 --parameters access=${access} \
 --parameters appsPort=${appsPort}

## エンドポイントURLの取得
az deployment group show \
 -g ${group_name} \
 -n deployPrj01 \
 --query properties.outputs.acaUrl.value
```

```
gunicorn --bind 0.0.0.0:5000 app:app -c app.py 
```

