# SOAP-Client-API-Development-with-Validation-and-Service-Hosting-
connecting to an actual public SOAP web service instead of just working with your own data

from flask import Flask, request, jsonify
from zeep import Client
import pycountry

app = Flask(__name__)

# Public CountryInfo SOAP WSDL
WSDL_URL = (
    "http://webservices.oorsprong.org/"
    "websamples.countryinfo/CountryInfoService.wso?WSDL"
)


# Validate the country code before contacting the SOAP service
def validate_country_code(country_code):

    if not isinstance(country_code, str):
        raise ValueError("Country code must be text.")

    country_code = country_code.strip()

    if len(country_code) != 2:
        raise ValueError("Country code must contain exactly two letters.")

    if country_code != country_code.upper():
        raise ValueError("Country code must use uppercase letters.")

    if not country_code.isalpha():
        raise ValueError("Country code must contain letters only.")

    # Check whether the code is a recognized ISO country code
    country = pycountry.countries.get(alpha_2=country_code)

    if country is None:
        raise ValueError("Country code is not recognized.")

    return country_code


# Connect to the SOAP service and retrieve a country's capital
def get_capital(country_code):

    try:
        # Generate a SOAP client from the public WSDL
        client = Client(WSDL_URL)

        # Call the CapitalCity SOAP operation
        result = client.service.CapitalCity(country_code)

        return {
            "country_code": country_code,
            "capital_city": result
        }

    except Exception as exc:
        # Provide a custom error message with the original exception
        raise RuntimeError(
            f"SOAP service request failed. Details: {exc}"
        ) from exc


# GET endpoint: Retrieve a capital using a query parameter
@app.route('/capital', methods=['GET'])
def capital_get():

    try:
        country_code = request.args.get('country_code')

        if not country_code:
            raise ValueError("Please provide a country_code parameter.")

        country_code = validate_country_code(country_code)

        result = get_capital(country_code)

        return jsonify(result), 200

    except ValueError as exc:
        return jsonify({
            "error": f"Invalid input: {exc}"
        }), 400

    except RuntimeError as exc:
        return jsonify({
            "error": str(exc)
        }), 502


# POST endpoint: Retrieve a capital using JSON input
@app.route('/capital', methods=['POST'])
def capital_post():

    try:
        data = request.get_json(silent=True)

        if not isinstance(data, dict):
            raise ValueError("Request must contain valid JSON.")

        if 'country_code' not in data:
            raise ValueError("Missing country_code field.")

        country_code = validate_country_code(data['country_code'])

        result = get_capital(country_code)

        return jsonify(result), 200

    except ValueError as exc:
        return jsonify({
            "error": f"Invalid input: {exc}"
        }), 400

    except RuntimeError as exc:
        return jsonify({
            "error": str(exc)
        }), 502


# Start the Flask development server
if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=True)
mkdir SOAP-API-PUBLIC-FETCH
cd SOAP-API-PUBLIC-FETCH
python -m venv venv
pip install Flask zeep pycountry
pip freeze > requirements.txt
