# Awesome Air-Gapped Kubernetes [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> 인터넷이 차단된 폐쇄망 환경에서 Kubernetes를 설치하고 운영하기 위한 도구, 함정, 가이드 모음.

모든 도구 항목에는 *폐쇄망* 메모가 붙어 있습니다. 오프라인 사용을 공식 지원하는지, 필요한 이미지·바이너리를 어떻게 뽑아내는지, 기본 설정에서 어떤 외부 호출을 하는지, 주기적으로 갱신해야 하는 데이터가 무엇인지를 각 프로젝트의 공식 문서나 저장소를 근거로 적었습니다.

[English](README.md)

## 목차

- [클러스터 설치](#클러스터-설치)
- [이미지 및 아티팩트 전송](#이미지-및-아티팩트-전송)
- [컨테이너 레지스트리](#컨테이너-레지스트리)
- [OS 및 언어 패키지 미러](#os-및-언어-패키지-미러)
- [오프라인 Helm 차트](#오프라인-helm-차트)
- [업그레이드와 수명 주기](#업그레이드와-수명-주기)
- [보안과 공급망](#보안과-공급망)
- [빌드와 배포](#빌드와-배포)
- [관측성](#관측성)
- [백업과 재해 복구](#백업과-재해-복구)
- [함정](#함정)
- [가이드, 발표, 블로그 글](#가이드-발표-블로그-글)

## 클러스터 설치

- [Kubespray](https://github.com/kubernetes-sigs/kubespray) - 프로덕션용 Kubernetes 클러스터를 배포하는 Ansible 플레이북. *폐쇄망:* `contrib/offline/generate_list.sh`가 필요한 파일·이미지 목록 전체를 만들고, 보조 스크립트가 이를 로컬 웹 서버와 레지스트리에 올리며, `files_repo`·`registry_host` 같은 인벤토리 변수로 모든 다운로드 경로를 바꿀 수 있습니다.
- [RKE2](https://docs.rke2.io/install/airgap) - Rancher의 보안 중심 Kubernetes 배포판. *폐쇄망:* 릴리스마다 이미지 tarball(`rke2-images.<arch>.tar.zst`)을 제공하며, `/var/lib/rancher/rke2/agent/images/`에 넣거나 사설 레지스트리에 올려 쓰고, 설치 스크립트는 로컬 아티팩트 경로를 받습니다.
- [K3s](https://docs.k3s.io/installation/airgap) - 단일 바이너리 경량 Kubernetes 배포판. *폐쇄망:* 릴리스마다 `k3s-airgap-images-<arch>.tar.zst` 이미지 tarball을 제공하고 `INSTALL_K3S_SKIP_DOWNLOAD=true`로 오프라인 설치하며, 노드에는 블랙홀 경로라도 기본 경로(default route)가 있어야 합니다.
- [k0s](https://github.com/k0sproject/k0s) - 의존성 없는 단일 바이너리 Kubernetes 배포판. *폐쇄망:* `k0s airgap list-images`로 필요한 이미지를 출력하고 `k0s airgap bundle-artifacts`로 번들을 만들며, 텔레메트리가 기본으로 켜져 있으므로 `spec.telemetry.enabled: false`로 끕니다.
- [Talos Linux](https://github.com/siderolabs/talos) - Kubernetes 전용 불변(immutable) API 관리형 Linux. *폐쇄망:* 레지스트리 미러와 부팅 미디어에 내장하는 이미지 캐시를 지원하지만, 기본값이 공개 디스커버리 서비스(`discovery.talos.dev`)를 쓰고 업그레이드는 Image Factory에 의존하므로 둘 다 온프레미스 대체가 필요합니다.
- [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/) - Kubernetes 컨트롤 플레인을 부트스트랩하는 업스트림 도구. *폐쇄망:* 공식 문서에 인터넷 없이 실행하는 절차가 있으며, `imageRepository`와 고정된 `kubernetesVersion`을 지정해 `kubeadm config images list/pull`로 이미지를 확인하고 미리 받습니다.
- [KubeKey](https://github.com/kubesphere/kubekey) - Kubernetes와 KubeSphere를 설치하는 Go 기반 설치 도구. *폐쇄망:* `kk artifact export`가 바이너리·이미지·선택적 OS 패키지를 하나의 tarball로 묶고 `kk create cluster -a`로 설치하며, 기본 다운로드 경로는 Google 스토리지입니다.
- [Kubean](https://github.com/kubean-io/kubean) - Kubespray 기반 클러스터 수명 주기 관리 오퍼레이터. *폐쇄망:* 릴리스마다 OS 패키지, 컨테이너 이미지, 바이너리 오프라인 패키지를 게시합니다.
- [MicroK8s](https://canonical.com/microk8s/docs/install-offline) - Canonical의 snap 패키지 Kubernetes. *폐쇄망:* `snap download`와 `snap ack`를 이용한 오프라인 설치가 문서화되어 있고, 이미지는 `microk8s images import`로 넣거나 `hosts.toml`에 지정한 미러에서 받습니다.

## 이미지 및 아티팩트 전송

- [Zarf](https://github.com/zarf-dev/zarf) - 폐쇄망 클러스터에 Kubernetes 앱을 전달하는 선언형 패키지 관리자. *폐쇄망:* 이미지·차트·파일을 담은 자체 완결형 `zarf-package-*.tar.zst` 아카이브를 만들며, 정적 컴파일된 단일 바이너리로 배포됩니다.
- [Hauler](https://github.com/hauler-dev/hauler) - 이미지·차트·파일을 OCI 아티팩트로 이식 가능한 저장소에 모으는 도구. *폐쇄망:* `hauler store save`가 `haul.tar.zst` 아카이브 하나를 만들고, 반대편에서 `hauler store load`와 `hauler store serve registry`로 복원·제공하며 서명과 SBOM도 유지합니다.
- [skopeo](https://github.com/podman-container-tools/skopeo) - 데몬 없이 컨테이너 이미지를 조회·복사·동기화하는 CLI. *폐쇄망:* `skopeo sync`로 저장소 전체를 디렉터리(예: 이동식 미디어)에 복사한 뒤 내부 레지스트리로 다시 올릴 수 있으며, README에 이 용도가 명시되어 있습니다.
- [crane](https://github.com/google/go-containerregistry/tree/main/cmd/crane) - 원격 이미지와 레지스트리를 다루는 Go CLI. *폐쇄망:* `crane pull`로 이미지를 tarball이나 OCI 레이아웃으로 저장하고 `crane push`로 내부 레지스트리에 올립니다.
- [ORAS](https://github.com/oras-project/oras) - OCI 아티팩트를 push·pull·copy하는 CLI. *폐쇄망:* `oras cp --to-oci-layout`과 `--from-oci-layout`으로 이미지와 임의 아티팩트를 디스크의 OCI 레이아웃을 거쳐 옮깁니다.
- [regclient](https://github.com/regclient/regclient) - `regctl`·`regsync` CLI를 포함한 레지스트리 클라이언트 라이브러리. *폐쇄망:* `regctl image export/import`로 tarball을 다루고, `regsync`는 OCI 레이아웃과의 미러링을 지원하며 문서에 폐쇄망 미러 유지 용도가 명시되어 있습니다.
- [imgpkg](https://github.com/carvel-dev/imgpkg) - 설정과 참조 이미지를 재배치 가능한 OCI 번들로 묶는 도구. *폐쇄망:* 공식 폐쇄망 절차가 있으며, `imgpkg copy --to-tar`로 번들과 모든 이미지를 내보내고 `imgpkg copy --tar --to-repo`로 가져옵니다.
- [oc-mirror](https://github.com/openshift/oc-mirror) - OpenShift 릴리스, 오퍼레이터, 차트, 이미지를 미러링하는 CLI. *폐쇄망:* `ImageSetConfiguration`을 기준으로 mirror-to-disk 모드가 tar 아카이브를 만들고 disk-to-mirror 모드가 폐쇄망 레지스트리에 올리며, v1은 지원 중단되었으므로 `--v2`를 씁니다.

## 컨테이너 레지스트리

- [Harbor](https://github.com/goharbor/harbor) - RBAC, 복제, 취약점 스캔을 갖춘 CNCF 졸업 레지스트리. *폐쇄망:* 오프라인 설치 패키지를 제공하지만 내장 Trivy 스캐너가 기본으로 DB를 내려받으므로, `skip_update: true`와 `offline_scan: true`를 설정하고 DB를 직접 넣거나 `db_repository`를 내부 미러로 지정합니다.
- [Zot](https://github.com/project-zot/zot) - 단일 바이너리로 제공되는 OCI 네이티브 레지스트리. *폐쇄망:* sync 확장으로 태그 필터를 걸어 주기적 또는 요청 시 미러링하며, CVE 스캔은 `dbRepository`를 내부 미러로 바꾸지 않으면 `updateInterval`마다 Trivy DB를 내려받습니다.
- [Distribution](https://github.com/distribution/distribution) - OCI 배포 레지스트리의 참조 구현. *폐쇄망:* 단순 내부 레지스트리로 쓸 수 있고, pull-through 캐시 모드는 업스트림 접근이 필요하며 인스턴스당 업스트림 하나만 프록시하므로 인터넷 연결 측에 둡니다.
- [Quay mirror-registry](https://github.com/quay/mirror-registry) - 소형 Project Quay 레지스트리 단독 설치 도구. *폐쇄망:* Quay 실행에 필요한 이미지를 모두 담은 오프라인 버전이 있고, Clair 취약점 데이터는 `clairctl export-updaters`와 `clairctl import-updaters`로 옮길 수 있습니다.

## OS 및 언어 패키지 미러

- [aptly](https://github.com/aptly-dev/aptly) - 미러·스냅샷·게시 기능을 갖춘 Debian/Ubuntu 저장소 관리자. *폐쇄망:* 스냅샷을 로컬 디렉터리에 게시하면 어떤 웹 서버로도 apt 클라이언트에 제공할 수 있습니다.
- [Pulp](https://github.com/pulp/pulpcore) - RPM, Debian, Python, 컨테이너 등을 다루는 플러그인 기반 저장소 관리자. *폐쇄망:* 내장 내보내기 기능이 저장소 버전을 체크섬과 함께 tar 아카이브로 만들어 네트워크가 격리된 Pulp로 가져가게 해 주지만, 내보내기·가져오기 경로를 명시적으로 허용해야 하고 양쪽 버전이 같아야 합니다.
- [reposync](https://dnf-plugins-core.readthedocs.io/en/latest/reposync.html) - 원격 RPM 저장소를 로컬 디렉터리로 복사하는 DNF 플러그인. *폐쇄망:* `--download-metadata`를 쓰면 복사본을 바로 저장소로 쓸 수 있지만, `--newest-only`와 함께 쓰면 받지 않은 패키지가 메타데이터에 남으므로 [createrepo_c](https://github.com/rpm-software-management/createrepo_c)로 다시 생성합니다.
- [bandersnatch](https://github.com/pypa/bandersnatch) - 파일 시스템이나 S3에 저장하는 PyPI 미러 클라이언트. *폐쇄망:* 허용 목록, 플랫폼, 최신 N개 릴리스, 크기 제한 같은 필터 플러그인으로 부분 미러를 작게 유지하고, 결과물은 정적 웹 서버로 제공합니다.
- [devpi](https://github.com/devpi/devpi) - PyPI 캐싱 프록시 겸 사설 패키지 인덱스. *폐쇄망:* `devpi-export`와 `devpi-import`로 사설 인덱스를 서버 간에 옮길 수 있지만 미러 캐시 파일은 내보내지 않으므로 미러 캐시는 옮길 수 없습니다.
- [Verdaccio](https://github.com/verdaccio/verdaccio) - 경량 사설 npm 레지스트리 겸 프록시. *폐쇄망:* `packages`의 `proxy` 항목을 모두 지우면 완전 오프라인이 되고, `publish.allow_offline: true`로 업링크 없이 게시할 수 있습니다.
- [Reposilite](https://github.com/dzikoysk/reposilite) - 경량 Maven 저장소 관리자. *폐쇄망:* 미러별로 `store`를 켜지 않으면 프록시한 아티팩트를 저장하지 않으며, `STRICT` 정책이 완전 오프라인 모드로 문서화되어 있습니다.
- [Athens](https://github.com/gomods/athens) - Go 모듈 프록시 서버. *폐쇄망:* `NetworkMode = "offline"`이면 저장된 모듈만 제공하며, 체크섬 DB 요청은 `NoSumPatterns`로 제외하지 않으면 기본으로 `sum.golang.org`에 전달됩니다.

## 오프라인 Helm 차트

- [Helm OCI registries](https://helm.sh/docs/topics/registries/) - 차트를 OCI 레지스트리에 저장하는 Helm 기본 기능. *폐쇄망:* `oci://` 참조로 `helm push`·`helm pull`을 하면 차트를 이미지와 같은 내부 레지스트리에 둘 수 있습니다.
- [ChartMuseum](https://github.com/helm/chartmuseum) - 저장소 백엔드를 바꿔 끼울 수 있는 Helm 차트 저장소 서버. *폐쇄망:* 로컬 파일 시스템 백엔드를 지원하므로 클라우드 스토리지 없이 실행됩니다.
- [helmfile](https://github.com/helmfile/helmfile) - 여러 Helm 릴리스를 선언형으로 배포하는 도구. *폐쇄망:* `helmfile fetch --output-dir`로 차트를 모두 로컬에 받으며 문서에 폐쇄망 용도가 명시되어 있고, `--skip-deps`로 배포 시점의 저장소 갱신을 건너뜁니다.
- [helm-images](https://github.com/nikhilsbhat/helm-images) - 차트나 릴리스가 배포하는 이미지를 나열하는 Helm 플러그인. *폐쇄망:* `helm images get`이 주어진 values 기준의 이미지 목록을 출력하므로 미러링 도구 입력으로 쓸 수 있습니다.

## 업그레이드와 수명 주기

- [Cluster API](https://github.com/kubernetes-sigs/cluster-api) - 클러스터를 생성·관리하는 Kubernetes 방식의 선언형 API. *폐쇄망:* `clusterctl`은 이미지 저장소 재지정(폐쇄망 용도로 문서화), 로컬 프로바이더 오버라이드, 로컬 cert-manager URL을 지원하며, `CLUSTERCTL_DISABLE_VERSIONCHECK`를 설정하지 않으면 실행할 때마다 새 버전을 확인합니다.
- [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller) - `Plan` 리소스로 노드를 업그레이드하는 Kubernetes 네이티브 컨트롤러. *폐쇄망:* 업그레이드 이미지를 미러링해 K3s·RKE2 폐쇄망 업그레이드에 쓰며, channel은 HTTP로 조회되므로 plan에는 `channel` 대신 `version`을 지정합니다.
- [image-builder](https://github.com/kubernetes-sigs/image-builder) - Cluster API 노드 이미지를 만드는 Packer·Ansible 도구. *폐쇄망:* `disable_public_repos`와 `extra_repos`로 패키지 설치를 내부 미러로 돌리고, `additional_registry_images`로 이미지를 노드 이미지에 미리 넣습니다.
- [kured](https://github.com/kubereboot/kured) - OS가 필요로 할 때 노드를 안전하게 재부팅하는 DaemonSet. *폐쇄망:* 직접 설정한 알림 URL이나 Prometheus 엔드포인트 외에는 외부 호출을 하지 않습니다.
- [Metal3](https://github.com/metal3-io/ironic-image) - Ironic 기반의 Kubernetes·Cluster API용 베어메탈 프로비저닝. *폐쇄망:* Ironic 이미지가 배포 커널과 램디스크에 `file://` URL을 받으므로 프로비저닝 에이전트를 로컬에서 제공할 수 있습니다.

## 보안과 공급망

- [Trivy](https://github.com/aquasecurity/trivy) - 취약점, 잘못된 설정, 시크릿, SBOM 스캐너. *폐쇄망:* `trivy-db`·`trivy-java-db`·`trivy-checks`를 OCI 아티팩트로 미러링해 지정하거나 `--skip-db-update`와 `--offline-scan`으로 갱신을 건너뛰며, 텔레메트리와 버전 확인이 기본으로 켜져 있습니다(`--disable-telemetry`, `--skip-version-check`).
- [Trivy Operator](https://github.com/aquasecurity/trivy-operator) - Trivy를 실행하고 결과를 커스텀 리소스로 저장하는 Kubernetes 오퍼레이터. *폐쇄망:* 취약점 DB 기본 위치는 `mirror.gcr.io`이며, Helm 값 `trivy.dbRegistry`·`trivy.dbRepository`·`trivy.offlineScan`으로 바꿉니다.
- [Grype](https://github.com/anchore/grype) - 이미지와 SBOM용 취약점 스캐너. *폐쇄망:* `grype db import`로 DB 아카이브를 넣고 `db.auto-update`와 `check-for-app-update`를 끄며, DB가 5일보다 오래되면 스캔이 실패하므로 갱신하거나 `db.validate-age`를 끕니다.
- [Syft](https://github.com/anchore/syft) - 컨테이너 이미지와 파일 시스템용 SBOM 생성기. *폐쇄망:* 기본으로 애플리케이션 업데이트를 확인하며(`check-for-app-update`), 패키지 레지스트리를 조회하는 네트워크 보강 기능은 켜지 않으면 꺼져 있습니다.
- [Cosign](https://github.com/sigstore/cosign) - 컨테이너 이미지와 아티팩트의 서명·검증 도구. *폐쇄망:* `--trusted-root`를 주거나 미러로 초기화하지 않으면 공개 TUF CDN에서 Sigstore 신뢰 루트를 받으며, 키 기반 검증은 `--insecure-ignore-tlog`로 Rekor를 건너뛸 수 있습니다.
- [Sigstore policy-controller](https://github.com/sigstore/policy-controller) - Cosign 서명·증명 정책을 강제하는 어드미션 컨트롤러. *폐쇄망:* `TrustRoot` 리소스가 직렬화된 TUF 저장소나 자체 Sigstore 키를 받으며, 투명성 로그를 지정하지 않으면 공개 Rekor를 씁니다.
- [Notation](https://github.com/notaryproject/notation) - Notary Project 형식으로 OCI 아티팩트를 서명·검증하는 CLI. *폐쇄망:* 레지스트리 없이 로컬 OCI 레이아웃을 검증할 수 있고(실험 기능), OCSP·CRL 폐기 확인에는 네트워크가 필요하므로 폐쇄망에서는 신뢰 정책에서 revocation을 `skip`으로 둡니다.
- [Kyverno](https://github.com/kyverno/kyverno) - 이미지 검증 기능을 갖춘 Kubernetes 네이티브 정책 엔진. *폐쇄망:* 이미지 검증은 레지스트리에 접근해야 하며, Sigstore 검증은 `rekor.ignoreTlog`, `ctlog.ignoreSCT`, 공개 키 직접 지정, TUF 미러로 오프라인에 맞출 수 있습니다.
- [Falco](https://github.com/falcosecurity/falco) - 컨테이너와 호스트 런타임 위협 탐지. *폐쇄망:* 기본 modern eBPF 프로브는 드라이버 다운로드가 필요 없지만, Helm 차트가 기본으로 `falcoctl`을 실행해 공개 인덱스에서 룰을 받고 추적하므로 끄거나 자체 OCI 레지스트리로 지정합니다.

## 빌드와 배포

- [Flux](https://github.com/fluxcd/flux2) - 클러스터를 Git·OCI 소스와 동기화하는 GitOps 도구 모음. *폐쇄망:* 공식 폐쇄망 가이드가 있으며, `flux install --export`로 미러링할 이미지를 확인하고 `flux bootstrap --registry`로 컨트롤러를 내부 레지스트리에 연결합니다.
- [Argo CD](https://github.com/argoproj/argo-cd) - Kubernetes용 선언형 GitOps 지속 배포 도구. *폐쇄망:* 별도의 폐쇄망 가이드는 없으며, `install.yaml`이 참조하는 `quay.io`·`ghcr.io`·`public.ecr.aws` 이미지를 미러링하고 Kustomize로 경로를 바꿉니다.
- [kaniko](https://github.com/chainguard-forks/kaniko) - 아카이브된 데몬리스 Dockerfile 빌더를 Chainguard가 유지보수하는 포크. *폐쇄망:* `--registry-mirror`로 Docker Hub를 대체하고 `--skip-default-registry-fallback`으로 대체 경로를 막으며, 포크는 보안 수정만 하고 바이너리 없이 소스 릴리스만 냅니다.
- [BuildKit](https://github.com/moby/buildkit) - `docker build`의 기반인 동시성·캐시 효율 이미지 빌드 엔진. *폐쇄망:* `buildkitd.toml`에서 레지스트리별 미러, CA 인증서, 평문 HTTP 옵션을 지정합니다.
- [Tekton Operator](https://tekton.dev/docs/operator/airgapimageconfiguration/) - Tekton Pipelines와 관련 컴포넌트를 설치·관리하는 오퍼레이터. *폐쇄망:* `TEKTON_REGISTRY_OVERRIDE`가 관리하는 모든 이미지의 레지스트리를 바꾸고, 이미지별 환경 변수로 세밀하게 조정할 수 있습니다.
- [Gitea](https://github.com/go-gitea/gitea) - 패키지 레지스트리와 CI를 갖춘 자체 호스팅 Git 서비스. *폐쇄망:* `OFFLINE_MODE`(CDN·Gravatar 미사용)는 기본으로 켜져 있지만 업데이트 확인 기능이 기본으로 `dl.gitea.com`을 호출하므로 꺼야 합니다.
- [Woodpecker](https://github.com/woodpecker-ci/woodpecker) - 경량 컨테이너 네이티브 CI 엔진. *폐쇄망:* 기본 clone 플러그인 이미지가 Docker Hub에 있으므로 미러로 바꾸고 신뢰 목록에 다시 등록해야 하며, 버전 확인이 기본으로 켜져 있습니다(`WOODPECKER_DISABLE_VERSION_CHECK`).
- [ko](https://github.com/ko-build/ko) - Dockerfile이나 Docker 데몬 없이 Go 컨테이너 이미지를 빌드하는 도구. *폐쇄망:* 기본 베이스 이미지가 `cgr.dev/chainguard/static`이므로 `defaultBaseImage`를 내부 미러로 지정하며, `--tarball`과 `--local`로 push 없이 빌드합니다.

## 관측성

- [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) - Prometheus Operator, Prometheus, Alertmanager, Grafana, exporter를 묶은 Helm 차트. *폐쇄망:* 이미지가 여러 공개 레지스트리에서 오므로 `global.imageRegistry`로 바꾸고, 내장 Grafana가 기본으로 켜져 있으니 Grafana 설정도 함께 적용합니다.
- [Grafana](https://github.com/grafana/grafana) - 메트릭·로그·트레이스 대시보드와 시각화. *폐쇄망:* 사용량 보고, 업데이트 확인, 뉴스 피드, Gravatar, 외부 스냅샷, 온라인 플러그인 카탈로그가 기본으로 켜져 있으므로 `grafana.ini`에서 끄고 플러그인은 이미지나 플러그인 디렉터리에 넣어 배포합니다.
- [Loki](https://github.com/grafana/loki) - 수평 확장형 로그 수집 시스템. *폐쇄망:* 기본으로 `stats.grafana.org`에 사용량 보고를 보내므로 `analytics.reporting_enabled: false`로 끕니다.
- [Grafana Alloy](https://github.com/grafana/alloy) - Prometheus 파이프라인을 포함한 OpenTelemetry 컬렉터 배포판. *폐쇄망:* 기본으로 익명 사용량을 보고하므로 `--disable-reporting`으로 실행합니다.

## 백업과 재해 복구

- [Velero](https://github.com/velero-io/velero) - Kubernetes 리소스와 영구 볼륨의 백업·복원·마이그레이션. *폐쇄망:* 온프레미스 문서에 미러링할 Velero·플러그인·restore-helper 이미지가 나와 있으며, 망 내부에 S3 호환 오브젝트 스토리지가 필요합니다.
- [etcd snapshots](https://etcd.io/docs/v3.6/op-guide/recovery/) - Kubernetes 데이터 저장소의 내장 백업·복원. *폐쇄망:* 완전히 로컬에서 동작하며, `etcdctl snapshot save`로 스냅샷을 만들고 v3.6부터는 `etcdutl`로 복원합니다.
- [Longhorn](https://github.com/longhorn/longhorn) - S3·NFS 백업을 지원하는 분산 블록 스토리지. *폐쇄망:* 이미지 목록과 save/load 스크립트를 포함한 공식 폐쇄망 가이드가 있고, 업그레이드 확인 기능이 기본으로 공개 서버를 호출하므로 꺼야 합니다.

## 함정

특정 도구와 무관한 일반적인 함정입니다.

- [동결된 구 레지스트리](https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/) - `k8s.gcr.io`는 2023년에 동결되었고 새 이미지는 `registry.k8s.io`에만 올라가므로, 어느 쪽이든 하드코딩된 참조는 내부 미러로 바꿔야 합니다.
- [샌드박스(pause) 이미지](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) - 컨테이너 런타임은 매니페스트가 아닌 자체 설정(기본값 `registry.k8s.io`)에서 pause 이미지를 받으므로, 워크로드 이미지를 모두 미러링해도 파드가 시작되지 않을 수 있습니다.
- [containerd 레지스트리 미러](https://github.com/containerd/containerd/blob/main/docs/hosts.md) - 미러는 `config_path`와 레지스트리별 `hosts.toml`로 설정하고 사설 CA 인증서도 여기에 지정하며, 예전의 인라인 `registry.mirrors` 설정은 지원 중단되었습니다.
- [암묵적 `Always` pull 정책](https://kubernetes.io/docs/concepts/containers/images/) - `:latest` 태그나 태그 없는 이미지는 `imagePullPolicy: Always`가 기본이라 컨테이너를 시작할 때마다 레지스트리에 접속합니다.
- [인증서 만료](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/) - kubeadm이 발급한 리프 인증서는 1년짜리이고 업그레이드할 때만 자동 갱신되므로, 업그레이드하지 않는 클러스터는 수동으로 갱신해야 합니다.
- [노드 패키지 저장소](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) - kubelet·kubeadm·kubectl 패키지는 `pkgs.k8s.io`에서 받고 구 apt·yum 저장소는 동결되었으므로, 폐쇄망 노드에는 로컬 패키지 미러가 필요합니다.
- [업스트림 DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/) - CoreDNS는 기본으로 클러스터 외부 이름을 노드의 `/etc/resolv.conf`로 전달하므로, 닿을 수 있는 업스트림이 없으면 외부 조회가 타임아웃됩니다.
- [검색 도메인과 ndots](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) - 파드는 기본으로 `ndots:5`라서 짧은 외부 이름도 모든 클러스터 검색 도메인을 붙여 먼저 조회합니다.
- [로컬 시간 소스](https://chrony-project.org/doc/4.6/chrony.conf.html) - 공개 NTP를 쓸 수 없으면 내부 시간 서버를 두어야 하며, chrony의 `local` 지시어가 격리된 네트워크용입니다.
- [Go 모듈 프록시](https://go.dev/ref/mod) - `GOPROXY`와 `GOSUMDB`의 기본값이 `proxy.golang.org`와 `sum.golang.org`이므로, 빌드에는 내부 프록시, vendoring, 설정 변경 중 하나가 필요합니다.
- [Helm 차트 의존성](https://helm.sh/docs/topics/charts/) - 차트 의존성은 원격 저장소 URL을 참조하므로, 망을 넘기기 전에 `charts/` 디렉터리를 포함시키거나 의존성 경로를 바꿉니다.
- [런타임 다운로드](https://trivy.dev/docs/latest/advanced/air-gap/) - 일부 컴포넌트는 시작한 뒤에 DB, 드라이버, 룰, 플러그인을 내려받으므로 컨테이너 이미지뿐 아니라 런타임 다운로드도 차트마다 점검합니다.
- [ACME 챌린지](https://cert-manager.io/docs/configuration/acme/http01/) - ACME HTTP-01 검증은 공개 CA가 클러스터에 접근해야 하므로 내부 [CA issuer](https://cert-manager.io/docs/configuration/ca/)를 대신 씁니다.

## 가이드, 발표, 블로그 글

- [Bootstrap an Air Gapped Cluster With Kubeadm](https://kubernetes.io/blog/2023/10/12/bootstrap-an-air-gapped-cluster-with-kubeadm/) - kubeadm용 패키지·이미지·매니페스트 준비 과정을 따라가는 Kubernetes 공식 블로그 글(2023).
- [Flux Air-Gapped Installation](https://fluxcd.io/flux/installation/configuration/air-gapped/) - Flux 이미지를 미러링하고 오프라인으로 GitOps를 부트스트랩하는 공식 가이드.
- [Talos Air-Gapped Environments](https://docs.siderolabs.com/talos/v1.13/platform-specific-installations/air-gapped) - 격리된 네트워크에서의 레지스트리 미러, Image Factory, 디스커버리, NTP, DNS를 다루는 공식 가이드.
- [How to Make Your K8s Cluster Survive When It Has No Internet Access](https://www.youtube.com/watch?v=bw9WxF2N7vc) - 폐쇄망에 맞춰야 할 런타임·레지스트리·정책 컴포넌트를 훑는 CNCF 발표(2023).
- [Managing Kubernetes in Air Gap/Offline Environments](https://www.youtube.com/watch?v=BkzIl5YL1b0) - 대규모 오프라인 클러스터 운영 경험을 다룬 KubeCon 발표(2019).
- [No Internet - No Problem? Air-Gapped Kubernetes on Bare Metal](https://www.youtube.com/watch?v=mNgNy9-mH2A) - 베어메탈 폐쇄망 Kubernetes 운영을 다룬 Open Source Summit 발표(2025).
- [Offline but Not Blind](https://www.youtube.com/watch?v=Ith_TOa85Lw) - 외부 SaaS 없이 폐쇄망 Kubernetes의 관측성을 구성하는 CNCF 발표(2026).
- [Service Networking Within Air-Gapped Environments](https://www.youtube.com/watch?v=Zvg2JWvc_oY) - 폐쇄망 클러스터의 서비스 네트워킹을 다룬 CNCF 발표(2026).

## 기여하기

기여를 환영합니다. 먼저 [기여 가이드](CONTRIBUTING.md)를 읽어 주세요. 모든 *폐쇄망* 메모는 풀 리퀘스트에 근거 링크를 함께 적어야 합니다.
