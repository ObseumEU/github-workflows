# Build-and-push

This GitHub workflow automates the build and push process for Docker images.

## Workflow Trigger

The workflow triggers on a workflow call event and expects three input parameters:

- `image`: Specifies the name of the Docker image to build and push.
- `dockerfile`: Specifies the path to the Dockerfile to use for building the image.
- `context`: Specifies the path to the build context directory.

## Workflow Steps

1. Get current date and time and set it as an output.
    ```yaml
    - name: Get current date
      id: date
      run: echo "::set-output name=date::$(date +'%Y-%m-%dT%H-%M-%S')"
    ```
2. Check out the repository and its submodules using the provided access token.
    ```yaml
    - uses: actions/checkout@v2
      with:
        submodules: recursive
        token: ${{ secrets.ACCESS_TOKEN }}
    ```
3. Use the `docker/metadata-action` to generate a set of tags for the Docker image based on the specified version pattern.
    ```yaml
    - name: Docker meta
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: |
          ${{ inputs.image }}
        tags: |
          type=semver,pattern={{version}}
    ```
4. Set up QEMU and Docker Buildx.
    ```yaml
    - name: Set up QEMU
      uses: docker/setup-qemu-action@v2
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    ```
5. Log in to Docker Hub using the provided credentials.
    ```yaml
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USER }}
        password: ${{ secrets.DOCKER_PASS }}
    ```
6. Build and push the Docker image.
    ```yaml
    - name: Build and push
      uses: docker/build-push-action@v3
      with:
        context: ${{ inputs.context }}
        push: true
        file: ${{ inputs.dockerfile }}
        tags: ${{ inputs.image }}:latest,${{ inputs.image }}:${{ steps.date.outputs.date }}
        github: ${{ secrets.ACCESS_TOKEN }}
    ```
7. Tag and push the current repository code with a tag based on the current date and time.
    ```yaml
    - name: Tag and push current repo code
      run: |
        git tag ${{ steps.date.outputs.date }}
        git push origin ${{ steps.date.outputs.date }}
    ```
