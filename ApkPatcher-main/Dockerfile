FROM python:3.13-slim-trixie

ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    TERM=xterm \
    DATA_DIR=/data \
    APKPATCHER_TOOLS_DIR=/opt/apkpatcher-tools \
    APKPATCHER_DESKTOP_APKTOOL=1 \
    APKPATCHER_NO_UPDATE=1

# Java مطلوب لـ APKTool / APKEditor / ApkSig، و radare2 لباتش Flutter فقط
RUN apt-get update \
 && apt-get install -y --no-install-recommends default-jre-headless ca-certificates \
 && (apt-get install -y --no-install-recommends radare2 || echo "radare2 غير متوفر: باتش Flutter لن يعمل") \
 && rm -rf /var/lib/apt/lists/*

# بدائل أوامر Termux التي تستدعيها الأداة
COPY docker/termux-stub.sh /usr/local/bin/pkg
RUN chmod +x /usr/local/bin/pkg \
 && ln -s /usr/local/bin/pkg /usr/local/bin/termux-wake-lock \
 && ln -s /usr/local/bin/pkg /usr/local/bin/termux-wake-unlock

WORKDIR /src
COPY . /src
RUN pip install --no-cache-dir . -r webui/requirements.txt

# تنزيل أدوات الجافا مسبقاً (APKTool / APKEditor / ApkSig) أثناء البناء.
# إن فشل التنزيل هنا فستحاول الأداة تنزيلها عند أول تشغيل.
RUN python -c "from ApkPatcher.Utils.Files_Check import FileCheck; f=FileCheck(); f.Set_Path(); f.isEmulator(); f.F_D(); f.F_D_A()" || true

RUN useradd -m -u 1000 app \
 && mkdir -p /data \
 && chown -R app:app /data /opt/apkpatcher-tools \
 && chown -R app:app "$(python -c 'import ApkPatcher,os;print(os.path.dirname(ApkPatcher.__file__))')"

WORKDIR /src/webui
USER app
VOLUME /data
EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s \
  CMD python -c "import urllib.request;urllib.request.urlopen('http://127.0.0.1:8000/healthz')" || exit 1

# عامل واحد فقط (الحالة محفوظة في الذاكرة) مع عدة threads
CMD ["gunicorn", "-w", "1", "--threads", "8", "--timeout", "0", "-b", "0.0.0.0:8000", "app:app"]
