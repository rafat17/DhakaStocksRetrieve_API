from flask import Flask, request, jsonify, render_template
import firebase_admin
from firebase_admin import credentials, db

#Setting up our flask app here

app = Flask(__name__)
app.config["DEBUG"] = True

#Setting up our firebase credentials for access to the realtime database

cred = credentials.Certificate('path/to/the/custom/jsonfile')

#Initialize firebase realtime database with this code
firebase_admin.initialize_app(cred, {
	'databaseURL': 'https://dhakastocks-v3.firebaseio.com/'
})

#Setting up our month dictionary to help with filtering of the data according to the timeline requested

month_index = {
	'1': 'January',
	'2': 'February',
	'3': 'March',
	'4': 'April',
	'5': 'May',
	'6': 'June',
	'7': 'July',
	'8': 'August',
	'9': 'September',
	'10': 'October',
	'11': 'November',
	'12': 'December'
}

#Method to determine which month has been selected from the dictionary
#Switch statement equivalent

def month_selector(month):
	if month=='January': return '1'
	elif month == 'February': return '2'
	elif month == 'March': return '3'
	elif month == 'April': return '4'
	elif month == 'May': return '5'
	elif month == 'June': return '6'
	elif month == 'July': return '7'
	elif month == 'August': return '8'
	elif month == 'September': return '9'
	elif month == 'October': return '10'
	elif month == 'November': return '11'
	else: return '12'


#Setting up app routes for creating endpoints to the database info access in JSON format

@app.route('/home')
def home():
	return 'This is where to place the documentation html file'

@app.route('/api/v2/<year>/all', methods=['GET'])
def get_all(year):

	url = None
	ref = None
	results = None

	month_dict = {}

	company = request.args.get('company')
	filter_by = request.args.get('filter_by')

#enters this block when either one or many filters are specified !
	if filter_by:
		filter_by = filter_by.split('|')

		for i in range(1, len(month_index)+1):
			i_tostr = str(i)
			url = '/' + year + '/' + month_index[i_tostr]
			ref = db.reference(url)

			lists = []

			if company:
				snapshots = ref.order_by_child('trading_code').equal_to(company).get()
			else:
				snapshots = ref.get()

			if(snapshots == None):
				month_dict[month_index[i_tostr]] = lists
				continue

			for key,value in snapshots.items():
				each_company_data = {"date": value['date'], "trading_code": value['trading_code']}

				for j in range(0, len(filter_by)):
					each_company_data[filter_by[j]] = value[filter_by[j]]

				each_company_data_json = {key: each_company_data}
				lists.append(each_company_data_json)

			month_dict[month_index[i_tostr]] = lists

		return jsonify(month_dict)


#enters this block when no filters are specified
	for i in range(1, len(month_index)+1):
		i_tostr = str(i)
		url = '/' + year + '/' + month_index[i_tostr]
		ref = db.reference(url)

		lists = []

		if company:
			snapshots = ref.order_by_child('trading_code').equal_to(company).get()
		else:
			snapshots = ref.get()

		if(snapshots == None):
			month_dict[month_index[i_tostr]] = lists
			continue

		for key,value in snapshots.items():

			each_company_data = {}
			each_company_data['date'] = value['date']
			each_company_data['close_price'] = value['close_price']
			each_company_data['high_price'] = value['high_price']
			each_company_data['low_price'] = value['low_price']
			each_company_data['ltp'] = value['ltp']
			each_company_data['open_price'] = value['open_price']
			each_company_data['trading_code'] = value['trading_code']
			each_company_data['volume'] = value['volume']
			each_company_data['trade'] = value['trade']
			each_company_data['value'] = value['value']

			each_company_data_json = {key: each_company_data}

			lists.append(each_company_data_json)

		month_dict[month_index[i_tostr]] = lists

	return jsonify(month_dict)



#Retrieves the monthly data for all companies by single month
#Optionally, can also filter data by parameters
@app.route('/api/v2/<year>/<month>', methods=['GET'])
def get_by_month(year, month):

	url =  '/'+ year + '/' + month
	ref = db.reference(url)

	filter_by = request.args.get('filter_by')
	company = request.args.get('company')

	if filter_by:
		filter_by = filter_by.split('|')

		if company:
			results_filtered = ref.order_by_child('trading_code').equal_to(company).get()
		else:
			results_filtered = ref.get()
		
		lists = []

		if(results_filtered == None):
			return jsonify({"error" : "404", "message" : "No results were founds"})

		for key,value in results_filtered.items():
			each_company_data = {"date": value['date'], "trading_code": value['trading_code']}

			for j in range(0, len(filter_by)):
				each_company_data[filter_by[j]] = value[filter_by[j]]

			each_company_data_json = {key: each_company_data}
			lists.append(each_company_data_json)

		return jsonify(lists)

		
	if(ref.get() == None):
		return "Invalid referencing or resource not available !"

	if company:
		results = ref.order_by_child('trading_code').equal_to(company).get()
	else:
		results = ref.get()

	lists = []

	for key, value in results.items():
		each_company_data = {}
		each_company_data['date'] = value['date']
		each_company_data['close_price'] = value['close_price']
		each_company_data['high_price'] = value['high_price']
		each_company_data['low_price'] = value['low_price']
		each_company_data['ltp'] = value['ltp']
		each_company_data['open_price'] = value['open_price']
		each_company_data['trading_code'] = value['trading_code']
		each_company_data['volume'] = value['volume']
		each_company_data['trade'] = value['trade']
		each_company_data['value'] = value['value']

		each_company_data_json = {key: each_company_data}

		lists.append(each_company_data_json)

	return jsonify(lists)



#API renders data according to the month length given in the HTTP request
#Optionally, can also filter data by parameters
@app.route('/api/v2/<year>/<startmonth>/to/<endmonth>', methods=['GET'])
def filter_by_timeline(year, startmonth, endmonth):
	
	startmonth_id = month_selector(startmonth)
	endmonth_id = month_selector(endmonth)

	filter_by = request.args.get('filter_by')
	company = request.args.get('company')
	print(company)

	if(int(startmonth_id) >= int(endmonth_id)):
		return 'Invalid referencing ! startmonth cannot be equal to or ahead of endmonth !'

	elif(int(endmonth_id) - int(startmonth_id) + 1 == 12):
		return 'Use /all at the end of api/v2/ request to get all data for a single year' 

	month_dict = {}

	if filter_by:
		filter_by = filter_by.split('|')

		for i in range(int(startmonth_id), int(endmonth_id)+1):
			lists = []
			url = '/' + year + '/' + month_index[str(i)]

			if company:
				results_filtered = db.reference(url).order_by_child('trading_code').equal_to(company).get()
			
			else:
				results_filtered = db.reference(url).get()

			if(results_filtered == None):
				return jsonify({'error' : '404', 'message' : 'No results found !'})

			for key,value in results_filtered.items():

				each_company_data = {"date" : value["date"], "trading_code" : value["trading_code"]}

				for j in range(0, len(filter_by)):
					each_company_data[filter_by[j]] = value[filter_by[j]]

				each_company_data_json = {key : each_company_data}
				lists.append(each_company_data_json)

			month_dict[month_index[str(i)]] = lists

		return jsonify(month_dict)



	for i in range(int(startmonth_id), int(endmonth_id)+1):
		lists = []
		url = '/' + year + '/' + month_index[str(i)]

		if company:
			results = db.reference(url).order_by_child('trading_code').equal_to(company).get()
		else:
			results = db.reference(url).get()

		if(results == None):
			return jsonify({'error' : '404', 'message' : 'No results found !'})

		for key, value in results.items():
			each_company_data = {}
			each_company_data['date'] = value['date']
			each_company_data['close_price'] = value['close_price']
			each_company_data['high_price'] = value['high_price']
			each_company_data['low_price'] = value['low_price']
			each_company_data['ltp'] = value['ltp']
			each_company_data['open_price'] = value['open_price']
			each_company_data['trading_code'] = value['trading_code']
			each_company_data['volume'] = value['volume']
			each_company_data['trade'] = value['trade']
			each_company_data['value'] = value['value']

			each_company_data_json = {key: each_company_data}

			lists.append(each_company_data_json)

		month_dict[month_index[str(i)]] = lists

	return jsonify(month_dict)



#Figure out a single entry by ID only
@app.route('/api/v2/<year>/id/<data_id>', methods=['GET'])
def filter_by_id(year, data_id):

	url  = '/' + year

	year_code = data_id[0:2]
	year = '20' + year_code
	month_code = data_id[2:4]
	date = data_id[4:6]

	month_id = month_code
	print(month_id)

	if(int(month_code) < 10):
		month_id = month_code[1]
		print(month_id)

	month = month_index[month_id]
	url += '/' + month + '/' + data_id
	ref = db.reference(url)

	if(ref.get() == None):
		return jsonify({'error' : '404', 'message' : 'No results found !'})

	else:
		return jsonify(ref.get())


	


if __name__ == '__main__':
	app.run()





	


