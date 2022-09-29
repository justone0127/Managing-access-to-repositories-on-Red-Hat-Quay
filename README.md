# Managing-access-to-repositories-on-Red-Hat-Quay

Red Hat Quay 사용자는 자신의 리포지토리를 생성하고 Red Hat Quay 인스턴스의 다른 사용자가 액세스할 수 있도록 할 수 있습니다. 또는 팀을 기반으로 하는 리포지토리에 대한 액세스를 허용하는 조직을 만들 수 있습니다. 또는 팀을 기반으로 하는 리포지토리에 대한 엑세스를 허용하는 조직을 만들 수 있습니다. 사용자 및 조직 리포지토리 모두에서 로봇 계정과 연결된 자격 증명을 생성하여 해당 리포지토리에 대한 엑세스를 허용할 수 있습니다. 로봇 계정을 사용하면 클라이언트에 Red Hat Quay 사용자 계정 없이도 다양한 컨테이너 클라이언트 (예: docker 또는 podman)가 저장소에 쉽게 액세스할 수 있습니다.



### 1. 사전 필수 사항

- Red Hat Quay 설치된 환경
- Container Image Registry에 Push



### 2. 사전 확인 및 조직 저장소에 대한 액세스 허용

Red Hat Quay는 조직을 만든 후에는 저장소 집합을 해당 조직에 직접 연결할 수 있습니다. 해당 조직의 리포지토리에 대한 액세스 권한을 추가하려면 Teams(동일한 권한을 가진 사용자 집합) 및 개별 사용자를 추가할 수 있습니다. 기본적으로 조직은 사용자와 동일한 리포지토리 및 로봇 계정 생성 기능을 가지고 있지만 조직은 사용자 그룹(팀 또는 개별)을 통해 공유 리포지토리를 설정하기 위한 것 입니다.



조직에 대해 알아야 할 기타 사항:

- 다른 조직에는 조직이 있을 수 없습니다. 조직을 세분화하려면 팀을 사용합니다.
- 조직은 사용자를 직접 포함할 수 없습니다. 먼저 팀을 추가한 다음 각 팀에 한 명 이상의 사용자를 추가해야 합니다. 
- 팀은 리포지토리 및 관련 이미지를 사용하는 구성원으로 또는 조직 관리를 위한 특별한 권한을 가진 관리자로 조직에서 설정할 수 있습니다.



1. OpenShift Container Platform에 Quay Registry Operator가 구성 및 배포되었는지 확인

2. Quay User 생성 확인

   - **devuser**
   - **opsuser**

3. Quay 콘솔 접속 > 조직 생성 : `devops`

   ![01_devops_org](https://github.com/justone0127/Managing-access-to-repositories-on-Red-Hat-Quay/blob/main/images/01_devops_org.png)

2. Repository 및 이미지 확인

   ![02_repository_image](https://github.com/justone0127/Managing-access-to-repositories-on-Red-Hat-Quay/blob/main/images/02_repository_image.png)

3. Team 생성 : `devteam`, `opsteam` 

   ![03_team](https://github.com/justone0127/Managing-access-to-repositories-on-Red-Hat-Quay/blob/main/images/03_team.png)

4. Team에 사용자 추가
   - `devteam`에는 `devuser` 를 멤버로 추가
   
  ![04_devuser](https://github.com/justone0127/Managing-access-to-repositories-on-Red-Hat-Quay/blob/main/images/04_devuser.png)
   
   - `opsteam`에는 `opsuesr`를 멤버로 추가
   
     ![05_opsuser](https://github.com/justone0127/Managing-access-to-repositories-on-Red-Hat-Quay/blob/main/images/05_opsuser.png)
   
5. Repository 권한 설정

   ![06_repository_permission](https://github.com/justone0127/Managing-access-to-repositories-on-Red-Hat-Quay/blob/main/images/06_repository_permission.png)

### 3. 액세스 권한에 따른 이미지 업로드 확인

1. 각 유저로 해당 이미지 저장소에 이미지 Pull & Push

   - `devuser`로 Image Push : 권한 없음 메시지를 확인할 수 있음

     - 로그인

       ```bash
       $ podman login -u devuser -p ${PASSWORD} ${QUAY_REGISTRY_HOST} --tls-verify=false
       Username: devuser
       Password:
       Login Succeeded!
       ```

     - 이미지 확인

       ```bash
       podman images |grep devops
       ${QUAY_REGISTRY}/devops/httpd-24-rhel7  2.4.203     87504a9170d1  13 days ago  332 MB
       
       ```
   
     - 이미지 푸시

       ```bash
   $ podman push ${QUAY_REGISTRY}/devops/httpd-24:latest --tls-verify=false
       Getting image source signatures
       Copying blob a95a08801800 [--------------------------------------] 8.0b / 20.0KiB
       Copying blob 80758011876c [--------------------------------------] 8.0b / 21.6MiB
   Copying blob 36656c1dceb1 [--------------------------------------] 8.0b / 86.0MiB
       Copying blob cf09fa03a4b3 [--------------------------------------] 8.0b / 205.7MiB
   Error: writing blob: initiating layer upload to /v2/devops/httpd-24/blobs/uploads/ in ${QUAY_REGISTRY}: unauthorized: access to the requested resource is not authorized
       ```
     
   - `opsuser`로 Image Push : 정상적으로 Push 됨
   
     - 이전 계정 로그아웃
   
       ```bash
       $ podman logout ${QUAY_REGISTRY}
       ```
   
     - `opsuser`로 로그인
   
       ```bash
       $ podman login -u opsuser -p  ${PASSWORD} ${QUAY_REGISTRY_HOST} --tls-verify=false
       Login Succeeded!
       ```
       
     - Image Push
     
       ```bash
    $ podman push ${QUAY_REGISTRY}/devops/httpd-24:latest --tls-verify=false
       Getting image source signatures
    Copying blob a95a08801800 done
       Copying blob 80758011876c done
       Copying blob 36656c1dceb1 done
       Copying blob cf09fa03a4b3 done
       Copying config 649628158f done
       Writing manifest to image destination
       Storing signatures
       ```
       
     - Image 확인
   
       ![07_image_upload](https://github.com/justone0127/Managing-access-to-repositories-on-Red-Hat-Quay/blob/main/images/07_image_upload.png)
