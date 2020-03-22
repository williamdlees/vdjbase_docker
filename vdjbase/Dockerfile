FROM vdjbase_base

MAINTAINER William Lees william@lees.org.uk

RUN cd \app && \
    git pull

COPY app/local_settings.py /app/website/.
COPY app/start.sh /app/.
COPY app/website.wsgi /app/.
COPY app/install_r_packages /app/.
COPY app/install_vdjbaseviz /app/.
COPY app/samples.zip /app/.

RUN unzip /app/samples.zip -d /app/uploads

CMD ["/app/start.sh"]
