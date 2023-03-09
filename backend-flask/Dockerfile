FROM python:3.10-slim-buster AS builder

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

FROM builder AS build1

COPY setup.sh /
RUN chmod +x /setup.sh

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "/setup.sh"]