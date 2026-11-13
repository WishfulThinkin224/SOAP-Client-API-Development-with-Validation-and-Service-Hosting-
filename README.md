# SOAP-Client-API-Development-with-Validation-and-Service-Hosting-
connecting to an actual public SOAP web service instead of just working with your own data
mkdir SOAP-API-PUBLIC-FETCH
cd SOAP-API-PUBLIC-FETCH
python -m venv venv
pip install Flask zeep pycountry
pip freeze > requirements.txt
