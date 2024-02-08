FROM 533267334695.dkr.ecr.ap-south-1.amazonaws.com/python3:slim-3.7
#using this ecr repo becouse code build has reached pull limit
#FROM python:3.7-slim
#FROM python:3.6-slim
#FROM python:3.7-slim -> use this if you are using the mysql as the db
WORKDIR /app
COPY . /app
RUN pip3 install -r requirements.txt
EXPOSE 5000
CMD ["python3","app_mysql.py"]
