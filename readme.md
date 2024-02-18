# k8s 교육

## 교육자료 github :  
    - doc 폴더 / k8s-기본교육-교재.pdf
  
## 실습  다운로드 : 
    - https://github.com/io203/k8s-edu.git
## gitOps 주소 : 
    - https://github.com/io203/demo-gitops.git


## sed 
```sh

## replace
cd ~/k8s-edu
find . -type f -name "*.yaml" -exec sed -i 's/$MASTER-1_EXTERNAL_IP/52.78.167.234/g' {} +
find . -type f -name "*.md" -exec sed -i 's/$MASTER-1_EXTERNAL_IP/52.78.167.234/g' {} +

cd ~/demo-gitops
find . -type f -name "*.yaml" -exec sed -i 's/$MASTER-1_EXTERNAL_IP/52.78.167.234/g' {} +

## 복원
cd ~/k8s-edu
find . -type f -name "*.yaml" -exec sed -i 's/52.78.167.234/$MASTER-1_EXTERNAL_IP/g' {} +
find . -type f -name "*.md" -exec sed -i 's/52.78.167.234/$MASTER-1_EXTERNAL_IP/g' {} +

cd ~/demo-gitops
find . -type f -name "*.yaml" -exec sed -i 's/52.78.167.234/$MASTER-1_EXTERNAL_IP/g' {} +



## 개별 폴더에서 할경우 
sed -i 's/$MASTER-1_EXTERNAL_IP/52.78.167.234/' *.yaml
sed -i 's/$MASTER-1_EXTERNAL_IP/52.78.167.234/' *.md
```
