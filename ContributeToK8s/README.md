# A Kind workflow for contributing to Kubernetes

**Slide**: http://kind.k8s.work

Kind: https://kind.sigs.k8s.io/docs/user/quick-start

Execute:

```bash
curl -L kind.k8s.work/fetch | bash
```

This will run:

```bash
docker run --rm -i -v /tmp:/tmp --network=host quay.io/kind-workshop/kind-fetch
```

Everything that is needed for the workshop.

After getting into `buildenv` container:

```bash
kind build node-image --image=local:master

```

