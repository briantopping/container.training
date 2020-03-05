FROM python:alpine
RUN pip install Flask
RUN pip install newrelic
COPY rng.py /
CMD ["python", "rng.py"]
EXPOSE 80
