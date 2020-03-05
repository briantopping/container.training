FROM python:alpine
RUN pip install redis
RUN pip install requests
RUN pip install newrelic
COPY worker.py /
CMD ["python", "worker.py"]
