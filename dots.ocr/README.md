```sh
docker run -d --restart unless-stopped \
  --name dots-ocr-container \
  -p 127.0.0.1:7880:8000 \
  -v /opt/ocr_tool/dots.ocr/weights/DotsOCR:/workspace/weights/DotsOCR \
  --env-file ./ocr.env \
  --gpus "device=0" \
  --entrypoint /bin/bash \
  dotsocr:latest \
  -c "
    set -ex;
    echo '--- Starting setup and server ---';
    echo 'Modifying vllm entrypoint...';
    sed -i '/^from vllm\.entrypoints\.cli\.main import main/a from DotsOCR import modeling_dots_ocr_vllm' \$(which vllm) && \
    echo 'vllm script after patch:' && \
    grep -A 1 'from vllm.entrypoints.cli.main import main' \$(which vllm) && \
    echo 'Starting server...' && \
    exec vllm serve /workspace/weights/DotsOCR \
        --tensor-parallel-size 1 \
        --gpu-memory-utilization 0.1 \
        --chat-template-content-format string \
        --served-model-name dotsocr-model \
        --trust-remote-code
  "
```
