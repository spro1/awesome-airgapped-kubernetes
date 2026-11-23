# Awesome Air-Gapped Kubernetes [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> Tools, gotchas, and guides for installing and operating Kubernetes without internet access.

Every tool entry carries an *Air-gap* note: whether the tool officially supports offline use, how to list or export what it needs, which outbound calls it makes by default, and what data it needs refreshed. Notes are based on each project's official documentation or repository.

[한국어](README.ko.md)

## Contents

- [Cluster Installation](#cluster-installation)
- [Image & Artifact Transfer](#image--artifact-transfer)
- [Container Registries](#container-registries)
- [OS & Language Package Mirrors](#os--language-package-mirrors)
- [Helm Charts Offline](#helm-charts-offline)
- [Upgrades & Lifecycle](#upgrades--lifecycle)
- [Security & Supply Chain](#security--supply-chain)
- [Build & Deploy](#build--deploy)
- [Observability](#observability)
- [Backup & DR](#backup--dr)
- [Gotchas](#gotchas)
- [Guides, Talks & Blog Posts](#guides-talks--blog-posts)

## Cluster Installation

- [Kubespray](https://github.com/kubernetes-sigs/kubespray) - Ansible playbooks for deploying production-ready Kubernetes clusters. *Air-gap:* `contrib/offline/generate_list.sh` produces the full file and image lists, helper scripts stage them on a local web server and registry, and inventory variables such as `files_repo` and `registry_host` redirect every download.
- [RKE2](https://docs.rke2.io/install/airgap) - Security-focused Kubernetes distribution from Rancher. *Air-gap:* Each release ships image tarballs (`rke2-images.<arch>.tar.zst`) that can be dropped into `/var/lib/rancher/rke2/agent/images/` or pushed to a private registry, and the install script accepts a local artifact path.
- [K3s](https://docs.k3s.io/installation/airgap) - Lightweight, single-binary Kubernetes distribution. *Air-gap:* Each release ships a `k3s-airgap-images-<arch>.tar.zst` image tarball, and the install script runs offline with `INSTALL_K3S_SKIP_DOWNLOAD=true`; nodes still need a default route, even a black-hole one.
- [k0s](https://github.com/k0sproject/k0s) - Single-binary, zero-dependency Kubernetes distribution. *Air-gap:* `k0s airgap list-images` prints the required images and `k0s airgap bundle-artifacts` builds a bundle; telemetry is enabled by default, so set `spec.telemetry.enabled: false`.
- [Talos Linux](https://github.com/siderolabs/talos) - Immutable, API-managed Linux distribution built for Kubernetes. *Air-gap:* Supports registry mirrors and an image cache baked into the boot media, but defaults use the public discovery service (`discovery.talos.dev`) and upgrades rely on Image Factory, so both need on-premises replacements.
- [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/) - Upstream tool for bootstrapping Kubernetes control planes. *Air-gap:* The docs cover running without internet: list and pre-pull control-plane images with `kubeadm config images list/pull`, using `imageRepository` and a pinned `kubernetesVersion`.
- [KubeKey](https://github.com/kubesphere/kubekey) - Go installer for Kubernetes and KubeSphere. *Air-gap:* `kk artifact export` packs binaries, images, and optional OS packages into one tarball that `kk create cluster -a` installs from; by default it downloads from Google storage.
- [Kubean](https://github.com/kubean-io/kubean) - Kubernetes operator for cluster lifecycle management built on Kubespray. *Air-gap:* Every release publishes offline packages for OS packages, container images, and binaries.
- [MicroK8s](https://canonical.com/microk8s/docs/install-offline) - Snap-packaged Kubernetes from Canonical. *Air-gap:* Documented offline install via `snap download` and `snap ack`, with images side-loaded through `microk8s images import` or pulled from a mirror set in `hosts.toml`.

## Image & Artifact Transfer

- [Zarf](https://github.com/zarf-dev/zarf) - Declarative package manager for delivering Kubernetes apps into disconnected clusters. *Air-gap:* Builds self-contained `zarf-package-*.tar.zst` archives with images, charts, and files, and ships as a single statically compiled binary.
- [Hauler](https://github.com/hauler-dev/hauler) - Collects images, charts, and files as OCI artifacts in a portable content store. *Air-gap:* `hauler store save` writes one `haul.tar.zst` archive, and `hauler store load` plus `hauler store serve registry` restores and serves it on the far side, keeping signatures and SBOMs.
- [skopeo](https://github.com/podman-container-tools/skopeo) - Daemonless CLI for inspecting, copying, and syncing container images. *Air-gap:* `skopeo sync` copies whole repositories to a directory (for example, removable media) and back into an internal registry, a use case named in its README.
- [crane](https://github.com/google/go-containerregistry/tree/main/cmd/crane) - Go CLI for interacting with remote images and registries. *Air-gap:* `crane pull` saves images as tarballs or OCI layouts and `crane push` uploads them to an internal registry.
- [ORAS](https://github.com/oras-project/oras) - CLI for pushing, pulling, and copying OCI artifacts. *Air-gap:* `oras cp --to-oci-layout` and `--from-oci-layout` move images and arbitrary artifacts through an on-disk OCI layout.
- [regclient](https://github.com/regclient/regclient) - Registry client library with the `regctl` and `regsync` CLIs. *Air-gap:* `regctl image export/import` handles tarballs, and `regsync` can mirror to or from an OCI layout, which its docs describe for maintaining a mirror over an air gap.
- [imgpkg](https://github.com/carvel-dev/imgpkg) - Packages configuration and its referenced images as relocatable OCI bundles. *Air-gap:* Has an official air-gapped workflow: `imgpkg copy --to-tar` exports a bundle with all its images and `imgpkg copy --tar --to-repo` imports it.
- [oc-mirror](https://github.com/openshift/oc-mirror) - OpenShift CLI for mirroring release payloads, operators, charts, and images. *Air-gap:* Driven by an `ImageSetConfiguration`, its mirror-to-disk mode writes a tar archive that disk-to-mirror loads into the disconnected registry; v1 is deprecated, so use `--v2`.

## Container Registries

- [Harbor](https://github.com/goharbor/harbor) - CNCF-graduated registry with RBAC, replication, and vulnerability scanning. *Air-gap:* Ships an offline installer, but the bundled Trivy scanner downloads its database by default; set `skip_update: true` and `offline_scan: true` and supply the database yourself, or point `db_repository` at an internal mirror.
- [Zot](https://github.com/project-zot/zot) - OCI-native registry delivered as a single binary. *Air-gap:* The sync extension mirrors periodically or on demand with tag filters, and CVE scanning downloads the Trivy database every `updateInterval` unless `dbRepository` points at an internal mirror.
- [Distribution](https://github.com/distribution/distribution) - Reference implementation of the OCI distribution registry. *Air-gap:* Works as a plain internal registry; its pull-through cache mode needs upstream access and proxies only one upstream per instance, so it belongs on the connected side.
- [Quay mirror-registry](https://github.com/quay/mirror-registry) - Standalone installer for a small Project Quay registry. *Air-gap:* An offline variant bundles every image needed to run Quay, and Clair vulnerability data can be moved across with `clairctl export-updaters` and `clairctl import-updaters`.

## OS & Language Package Mirrors

- [aptly](https://github.com/aptly-dev/aptly) - Debian and Ubuntu repository manager with mirrors, snapshots, and publishing. *Air-gap:* Publishes snapshots to a local directory that any web server can serve to apt clients.
- [Pulp](https://github.com/pulp/pulpcore) - Plugin-based repository manager for RPM, Debian, Python, container, and other content. *Air-gap:* Built-in export writes repository versions to tar archives with checksums for import into a network-isolated Pulp, but export and import paths must be allowed explicitly and both sides must run the same version.
- [reposync](https://dnf-plugins-core.readthedocs.io/en/latest/reposync.html) - DNF plugin that copies a remote RPM repository to a local directory. *Air-gap:* `--download-metadata` makes the copy usable as-is, but with `--newest-only` the metadata still lists skipped packages, so regenerate it with [createrepo_c](https://github.com/rpm-software-management/createrepo_c).
- [bandersnatch](https://github.com/pypa/bandersnatch) - PyPI mirror client that writes to a filesystem or S3. *Air-gap:* Filter plugins (allowlists, platforms, latest-N releases, size limits) keep partial mirrors small, and the result is served by any static web server.
- [devpi](https://github.com/devpi/devpi) - PyPI caching proxy and private package index. *Air-gap:* `devpi-export` and `devpi-import` move private indexes between servers, but cached mirror files are not exported, so a mirror cache cannot be carried across.
- [Verdaccio](https://github.com/verdaccio/verdaccio) - Lightweight private npm registry and proxy. *Air-gap:* Removing every `proxy` field from `packages` makes it fully offline, and `publish.allow_offline: true` allows publishing without an uplink.
- [Reposilite](https://github.com/dzikoysk/reposilite) - Lightweight Maven repository manager. *Air-gap:* Proxied artifacts are not stored unless `store` is enabled per mirror, and the `STRICT` policy is documented as full offline mode.
- [Athens](https://github.com/gomods/athens) - Go module proxy server. *Air-gap:* `NetworkMode = "offline"` serves only stored modules, and checksum database requests are forwarded to `sum.golang.org` by default unless excluded with `NoSumPatterns`.

## Helm Charts Offline

- [Helm OCI registries](https://helm.sh/docs/topics/registries/) - Helm's native support for storing charts in OCI registries. *Air-gap:* `helm push` and `helm pull` with `oci://` references let charts live in the same internal registry as their images.
- [ChartMuseum](https://github.com/helm/chartmuseum) - Helm chart repository server with pluggable storage. *Air-gap:* Supports a local filesystem storage backend, so it runs without any cloud storage.
- [helmfile](https://github.com/helmfile/helmfile) - Declarative specification for deploying sets of Helm releases. *Air-gap:* `helmfile fetch --output-dir` downloads all charts locally, documented as useful for air-gapped environments, and `--skip-deps` avoids repository updates at deploy time.
- [helm-images](https://github.com/nikhilsbhat/helm-images) - Helm plugin that lists the images a chart or release deploys. *Air-gap:* `helm images get` prints the image list for given values, ready to feed a mirroring tool.

## Upgrades & Lifecycle

- [Cluster API](https://github.com/kubernetes-sigs/cluster-api) - Declarative Kubernetes-style APIs for creating and managing clusters. *Air-gap:* `clusterctl` supports image repository overrides (documented for air-gapped use), local provider overrides, and a local cert-manager URL; it checks for new versions on every run unless `CLUSTERCTL_DISABLE_VERSIONCHECK` is set.
- [system-upgrade-controller](https://github.com/rancher/system-upgrade-controller) - Kubernetes-native node upgrade controller driven by `Plan` resources. *Air-gap:* Used for K3s and RKE2 air-gapped upgrades with mirrored upgrade images; set `version` instead of `channel` in plans, because channels are resolved over HTTP.
- [image-builder](https://github.com/kubernetes-sigs/image-builder) - Packer and Ansible tooling for building Cluster API node images. *Air-gap:* `disable_public_repos` with `extra_repos` switches package installs to internal mirrors, and `additional_registry_images` pre-bakes images into the node image.
- [kured](https://github.com/kubereboot/kured) - DaemonSet that safely reboots nodes when the OS requires it. *Air-gap:* Makes outbound calls only to a notification URL or Prometheus endpoint you configure.
- [Metal3](https://github.com/metal3-io/ironic-image) - Bare-metal host provisioning for Kubernetes and Cluster API, based on Ironic. *Air-gap:* The Ironic image accepts `file://` URLs for the deploy kernel and ramdisk, so the provisioning agent can be served locally.

## Security & Supply Chain

- [Trivy](https://github.com/aquasecurity/trivy) - Scanner for vulnerabilities, misconfigurations, secrets, and SBOMs. *Air-gap:* Mirror `trivy-db`, `trivy-java-db`, and `trivy-checks` as OCI artifacts and point Trivy at them, or skip updates with `--skip-db-update` and `--offline-scan`; telemetry and version checks are on by default (`--disable-telemetry`, `--skip-version-check`).
- [Trivy Operator](https://github.com/aquasecurity/trivy-operator) - Kubernetes operator that runs Trivy and stores results as custom resources. *Air-gap:* Helm values `trivy.dbRegistry`, `trivy.dbRepository`, and `trivy.offlineScan` redirect the vulnerability database, which defaults to `mirror.gcr.io`.
- [Grype](https://github.com/anchore/grype) - Vulnerability scanner for images and SBOMs. *Air-gap:* `grype db import` loads a database archive; disable `db.auto-update` and `check-for-app-update`, and note that scans fail on a database older than five days unless it is refreshed or `db.validate-age` is disabled.
- [Syft](https://github.com/anchore/syft) - SBOM generator for container images and filesystems. *Air-gap:* Checks for application updates by default (`check-for-app-update`), while network enrichment from package registries is off unless enabled.
- [Cosign](https://github.com/sigstore/cosign) - Signing and verification for container images and other artifacts. *Air-gap:* Fetches Sigstore trust roots from a public TUF CDN unless given `--trusted-root` or initialized from a mirror; key-based verification can skip Rekor with `--insecure-ignore-tlog`.
- [Sigstore policy-controller](https://github.com/sigstore/policy-controller) - Admission controller that enforces Cosign signature and attestation policies. *Air-gap:* The `TrustRoot` resource accepts a serialized TUF repository or your own Sigstore keys; an unset transparency log falls back to public Rekor.
- [Notation](https://github.com/notaryproject/notation) - CLI for signing and verifying OCI artifacts with the Notary Project format. *Air-gap:* Can verify a local OCI layout without a registry (experimental); OCSP and CRL revocation checks need network access, so disconnected setups set revocation to `skip` in the trust policy.
- [Kyverno](https://github.com/kyverno/kyverno) - Kubernetes-native policy engine with image verification. *Air-gap:* Image verification must reach your registry, and Sigstore checks can be adapted offline with `rekor.ignoreTlog`, `ctlog.ignoreSCT`, supplied public keys, or a TUF mirror.
- [Falco](https://github.com/falcosecurity/falco) - Runtime threat detection for containers and hosts. *Air-gap:* The default modern eBPF probe needs no driver download, but the Helm chart runs `falcoctl` by default to fetch and follow rules from public indexes, so disable it or point it at your own OCI registry.

## Build & Deploy

- [Flux](https://github.com/fluxcd/flux2) - GitOps toolkit for keeping clusters in sync with Git and OCI sources. *Air-gap:* Has an official air-gapped guide: `flux install --export` lists the images to mirror and `flux bootstrap --registry` points controllers at the internal registry.
- [Argo CD](https://github.com/argoproj/argo-cd) - Declarative GitOps continuous delivery for Kubernetes. *Air-gap:* No dedicated air-gap guide; `install.yaml` references images on `quay.io`, `ghcr.io`, and `public.ecr.aws` that need mirroring and rewriting with Kustomize.
- [kaniko](https://github.com/chainguard-forks/kaniko) - Chainguard's maintained fork of the archived daemonless Dockerfile builder. *Air-gap:* `--registry-mirror` replaces Docker Hub and `--skip-default-registry-fallback` stops fallback to it; the fork ships source releases only, with security fixes but no planned feature work.
- [BuildKit](https://github.com/moby/buildkit) - Concurrent, cache-efficient image build engine behind `docker build`. *Air-gap:* `buildkitd.toml` sets per-registry mirrors, CA certificates, and plain-HTTP options.
- [Tekton Operator](https://tekton.dev/docs/operator/airgapimageconfiguration/) - Operator that installs and manages Tekton Pipelines and related components. *Air-gap:* `TEKTON_REGISTRY_OVERRIDE` rewrites the registry of every managed image, with per-image environment variables for finer control.
- [Gitea](https://github.com/go-gitea/gitea) - Self-hosted Git forge with a package registry and CI. *Air-gap:* `OFFLINE_MODE` (no CDN or Gravatar) is on by default, but the update checker calls `dl.gitea.com` by default and should be disabled.
- [Woodpecker](https://github.com/woodpecker-ci/woodpecker) - Lightweight container-native CI engine. *Air-gap:* The default clone plugin image comes from Docker Hub and must be repointed to a mirror and trusted again; the version check is on by default (`WOODPECKER_DISABLE_VERSION_CHECK`).
- [ko](https://github.com/ko-build/ko) - Builds Go container images without a Dockerfile or Docker daemon. *Air-gap:* The default base image is `cgr.dev/chainguard/static`, so set `defaultBaseImage` to an internal mirror; `--tarball` and `--local` build without pushing.

## Observability

- [kube-prometheus-stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack) - Helm chart bundling Prometheus Operator, Prometheus, Alertmanager, Grafana, and exporters. *Air-gap:* Images come from several public registries and can be redirected with `global.imageRegistry`; the bundled Grafana is enabled by default, so apply its settings too.
- [Grafana](https://github.com/grafana/grafana) - Dashboards and visualization for metrics, logs, and traces. *Air-gap:* Usage reporting, update checks, the news feed, Gravatar, external snapshots, and the online plugin catalog are on by default; disable them in `grafana.ini` and ship plugins inside the image or plugins directory.
- [Loki](https://github.com/grafana/loki) - Horizontally scalable log aggregation system. *Air-gap:* Sends usage reports to `stats.grafana.org` by default; set `analytics.reporting_enabled: false`.
- [Grafana Alloy](https://github.com/grafana/alloy) - OpenTelemetry collector distribution with Prometheus pipelines. *Air-gap:* Reports anonymous usage by default; start it with `--disable-reporting`.

## Backup & DR

- [Velero](https://github.com/velero-io/velero) - Backup, restore, and migration of Kubernetes resources and persistent volumes. *Air-gap:* The on-premises docs list the Velero, plugin, and restore-helper images to mirror, and it needs an S3-compatible object store inside the network.
- [etcd snapshots](https://etcd.io/docs/v3.6/op-guide/recovery/) - Built-in backup and restore for the Kubernetes datastore. *Air-gap:* Fully local: `etcdctl snapshot save` takes the snapshot and, from v3.6, `etcdutl` restores it.
- [Longhorn](https://github.com/longhorn/longhorn) - Distributed block storage with backups to S3 or NFS. *Air-gap:* Has an official air-gap guide with an image list and save/load scripts; the upgrade checker calls a public server by default and should be disabled.

## Gotchas

General pitfalls that are independent of any tool.

- [Frozen legacy registry](https://kubernetes.io/blog/2023/02/06/k8s-gcr-io-freeze-announcement/) - `k8s.gcr.io` was frozen in 2023 and new images go only to `registry.k8s.io`, so hard-coded references to either must be rewritten to the internal mirror.
- [Sandbox (pause) image](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) - The container runtime pulls the pause image from its own configuration (`registry.k8s.io` by default), not from your manifests, so pods fail to start even when every workload image is mirrored.
- [containerd registry mirrors](https://github.com/containerd/containerd/blob/main/docs/hosts.md) - Mirrors are configured with `config_path` and per-registry `hosts.toml` files, which is also where private CA certificates go; the older inline `registry.mirrors` configuration is deprecated.
- [Implicit `Always` pull policy](https://kubernetes.io/docs/concepts/containers/images/) - Images tagged `:latest` or untagged default to `imagePullPolicy: Always`, so every container start contacts the registry.
- [Certificate expiry](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/) - Leaf certificates issued by kubeadm last one year and are renewed automatically only during upgrades, so a cluster that is never upgraded needs manual renewal.
- [Node package repositories](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) - kubelet, kubeadm, and kubectl packages come from `pkgs.k8s.io` and the legacy apt and yum repositories are frozen, so offline nodes need a local package mirror.
- [Upstream DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/) - CoreDNS forwards non-cluster names to the node's `/etc/resolv.conf` by default, so without a reachable upstream resolver external lookups time out.
- [Search domains and ndots](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) - Pods default to `ndots:5`, so short external names are tried against every cluster search domain before being resolved as-is.
- [Local time source](https://chrony-project.org/doc/4.6/chrony.conf.html) - Without public NTP, run an internal time server; chrony's `local` directive is intended for isolated networks.
- [Go module proxy](https://go.dev/ref/mod) - `GOPROXY` and `GOSUMDB` default to `proxy.golang.org` and `sum.golang.org`, so builds need an internal proxy, vendoring, or adjusted settings.
- [Helm chart dependencies](https://helm.sh/docs/topics/charts/) - Chart dependencies reference remote repository URLs, so vendor the `charts/` directory or repoint dependencies before crossing the gap.
- [Runtime downloads](https://trivy.dev/docs/latest/advanced/air-gap/) - Some components fetch databases, drivers, rules, or plugins after they start, so audit every chart for runtime downloads, not only for container images.
- [ACME challenges](https://cert-manager.io/docs/configuration/acme/http01/) - ACME HTTP-01 validation requires the public CA to reach the cluster, so use an internal [CA issuer](https://cert-manager.io/docs/configuration/ca/) instead.

## Guides, Talks & Blog Posts

- [Bootstrap an Air Gapped Cluster With Kubeadm](https://kubernetes.io/blog/2023/10/12/bootstrap-an-air-gapped-cluster-with-kubeadm/) - Kubernetes blog walkthrough of staging packages, images, and manifests for kubeadm (2023).
- [Flux Air-Gapped Installation](https://fluxcd.io/flux/installation/configuration/air-gapped/) - Official guide to mirroring Flux images and bootstrapping GitOps offline.
- [Talos Air-Gapped Environments](https://docs.siderolabs.com/talos/v1.13/platform-specific-installations/air-gapped) - Official guide covering registry mirrors, Image Factory, discovery, NTP, and DNS for isolated networks.
- [How to Make Your K8s Cluster Survive When It Has No Internet Access](https://www.youtube.com/watch?v=bw9WxF2N7vc) - CNCF talk surveying the runtime, registry, and policy components to adapt for air gaps (2023).
- [Managing Kubernetes in Air Gap/Offline Environments](https://www.youtube.com/watch?v=BkzIl5YL1b0) - KubeCon talk on operating offline clusters at scale (2019).
- [No Internet - No Problem? Air-Gapped Kubernetes on Bare Metal](https://www.youtube.com/watch?v=mNgNy9-mH2A) - Open Source Summit talk on running air-gapped Kubernetes on bare metal (2025).
- [Offline but Not Blind](https://www.youtube.com/watch?v=Ith_TOa85Lw) - CNCF talk on observability in air-gapped Kubernetes without external SaaS (2026).
- [Service Networking Within Air-Gapped Environments](https://www.youtube.com/watch?v=Zvg2JWvc_oY) - CNCF talk on service networking in air-gapped clusters (2026).

## Contributing

Contributions are welcome. Read the [contribution guidelines](CONTRIBUTING.md) first; every *Air-gap* note must link its evidence in the pull request.
