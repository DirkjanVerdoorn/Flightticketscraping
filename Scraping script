import json
import requests
from lxml import html
from collections import OrderedDict
import datetime 
import re

departure_date = "02-28-2019"
return_date = "03-18-2019"
url = "https://www.expedia.com/Flights-Search?trip=roundtrip&leg1=from:AMS,to:LAX,departure:"+departure_date+",TANYT&leg2=from:LAX,to:AMS,departure:"+return_date+",TANYT&passengers=adults:1,children:0,seniors:0,infantinlap:Y&options=cabinclass%3Aeconomy&mode=search&origref=www.expedia.com"
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.77 Safari/537.36'}
response = requests.get(url, headers=headers)
parser = html.fromstring(response.text)
json_data_xpath = parser.xpath("//script[@id='cachedResultsJson']//text()")
raw_json =json.loads(json_data_xpath[0] if json_data_xpath else'')
flight_data = json.loads(raw_json["content"])

filename = "Flights " + "_" + str(datetime.datetime.now().date()) + ".csv"
f = open(filename, "w")

headers = "Price, Departure Location Airpot, Arrival Location Airport, Airline, Plane, Stops, Plane Code, Departure Time, Arrival Time, Date of Scrape, Days until departure\n"
f.write(headers)

for i in flight_data['legs'].keys():
	exact_price = flight_data['legs'][i].get('price',{}).get('totalPriceAsDecimal','')

	departure_location_airport = flight_data['legs'][i].get('departureLocation',{}).get('airportLongName','')
	departure_location_city = flight_data['legs'][i].get('departureLocation',{}).get('airportCity','')
	
	arrival_location_airport = flight_data['legs'][i].get('arrivalLocation',{}).get('airportLongName','')
	arrival_location_city = flight_data['legs'][i].get('arrivalLocation',{}).get('airportCity','')
	
	airline_name = flight_data['legs'][i].get('carrierSummary',{}).get('airlineName','')
	carrier = flight_data['legs'][i].get('timeline',[])[0].get('carrier',{})

	if not airline_name:
		airline_name = "Undefined"

	departure = departure_location_airport+", "+departure_location_city
	arrival = arrival_location_airport+", "+arrival_location_city

	for timeline in  flight_data['legs'][i].get('timeline',{}):
		if 'departureAirport' in timeline.keys():
				rep = {"uur": "", ".": ":"}
				rep = dict((re.escape(k), v) for k, v in rep.items())
				pattern = re.compile("|".join(rep.keys()))

				departure_time = timeline['departureTime'].get('time','')
				clean_departure_time = pattern.sub(lambda m: rep[re.escape(m.group(0))], departure_time)
				arrival_time = timeline.get('arrivalTime',{}).get('time','')
				clean_arrival_time = pattern.sub(lambda m: rep[re.escape(m.group(0))], arrival_time)

				clean_departure_time = str(clean_departure_time)
				clean_arrival_time = str(clean_arrival_time)

	plane = carrier.get('plane','')
	if not plane:
		plane = "Undefined"
	plane_code = carrier.get('planeCode','')
	if not plane_code:
		plane_code = "Undefined"
	price = "{0:.2f}".format(exact_price)
	if not price:
		price = "Undefined"

	no_of_stops = flight_data['legs'][i].get("stops","")

	if no_of_stops==0:
		stop = "Nonstop"
	elif no_of_stops==1:
		stop = str(no_of_stops)+' Stop'
	else:
		stop = str(no_of_stops)+' Stops'

	price = str(price)
	departure_location_city = str(departure_location_city)
	airline_name = str(airline_name)	
	plane = str(plane)
	stop = str(stop)
	plane_code = str(plane_code)

	departure_date_clean = re.sub("-", " ", departure_date)
	datetime_object = datetime.datetime.strptime(departure_date_clean, '%m %d %Y')

	date_of_scrape = datetime.datetime.now().strftime('%m %d %Y')
	datescrape_object = datetime.datetime.strptime(date_of_scrape, '%m %d %Y')

	time_until_departure = datetime_object - datescrape_object
	time_until_departure_object = str(time_until_departure)
	time_until_departure_object_clean = re.sub("days, 0:00:00", "", time_until_departure_object)

	date_dis = str(datetime.datetime.now().date())
	f.write(price + "," + departure_location_city + "," + arrival_location_city + "," + airline_name + "," + plane + "," + stop + "," + plane_code + "," 
		+ clean_departure_time + "," + clean_arrival_time + "," + date_dis + "," + time_until_departure_object_clean + "\n")

f.close()

