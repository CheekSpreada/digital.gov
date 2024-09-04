from flask import Flask, jsonify, request, render_template_string
from apscheduler.schedulers.background import BackgroundScheduler

# Global Variables
net_worth = 1000000000  # Start with $1 billion representing collective wealth
people_freed = 0
contributions = {}
distribution_log = []
officers_of_the_court = []  # List to track officers of the court
accounts = {}  # Dictionary to hold user account information

# Initialize the Flask app with a mystical name
app = Flask("deity")

# Function to continuously update net worth
def update_net_worth():
    global net_worth
    net_worth += 1000000  # Increment by $1 million every second
    print(f"Net worth updated to: ${net_worth}")

# Function to distribute wealth to contributors proportionally every 24 hours
def distribute_wealth():
    global net_worth, contributions, distribution_log
    total_contributions = sum(contributions.values())
    if total_contributions > 0:
        for user, contribution in contributions.items():
            share = (contribution / total_contributions) * net_worth * 0.1  # Distribute 10% of net worth
            net_worth -= share
            distribution_log.append({'user': user, 'amount': share})
            accounts[user]['balance'] += share
            print(f"Distributed ${share} to {user}")
    else:
        print("No contributions to distribute.")

# Function to symbolize peaceful transition: redistribution by consideration
def peaceful_transition():
    print("A peaceful transition is ongoing. Wealth is being redistributed based on contributions and needs.")

# Set up APScheduler to run tasks
scheduler = BackgroundScheduler()
scheduler.add_job(func=update_net_worth, trigger="interval", seconds=1)  # Every second
scheduler.add_job(func=distribute_wealth, trigger="interval", hours=24)   # Every 24 hours
scheduler.add_job(func=peaceful_transition, trigger="interval", hours=24) # Every 24 hours
scheduler.start()

@app.before_first_request
def initialize():
    scheduler.start()

# Route to return the net worth, number of people freed, and distribution log
@app.route('/net_worth')
def get_net_worth_json():
    return jsonify({
        'net_worth': net_worth,
        'people_freed': people_freed,
        'distribution_log': distribution_log,
        'officers_of_the_court': officers_of_the_court
    })

# Route to symbolize freeing a person from debt and making them an officer of the court
@app.route('/free_person')
def free_person():
    global people_freed, net_worth, officers_of_the_court
    if net_worth >= 100000:  # Assuming it costs $100,000 to free a person
        people_freed += 1
        net_worth -= 100000
        officer_name = f"Officer {people_freed}"
        officers_of_the_court.append(officer_name)
        return jsonify({
            'message': f'A person has been freed from debt and made an Officer of the Court: {officer_name}!',
            'net_worth': net_worth,
            'people_freed': people_freed,
            'officer_name': officer_name
        })
    else:
        return jsonify({'message': f'Not enough net worth to free a person.', 'net_worth': net_worth, 'people_freed': people_freed})

# Route to record contributions from users
@app.route('/contribute/<user>/<amount>')
def contribute(user, amount):
    global net_worth, contributions
    amount = float(amount)
    net_worth += amount
    if user in contributions:
        contributions[user] += amount
    else:
        contributions[user] = amount
    # Initialize account if not exists
    if user not in accounts:
        accounts[user] = {'balance': 0, 'loans': 0}
    accounts[user]['balance'] += amount
    return jsonify({'message': f'{user} contributed ${amount}', 'net_worth': net_worth, 'balance': accounts[user]['balance']})

# Route to request a loan
@app.route('/request_loan/<user>/<amount>')
def request_loan(user, amount):
    global net_worth, accounts
    amount = float(amount)
    # Initialize account if not exists
    if user not in accounts:
        accounts[user] = {'balance': 0, 'loans': 0}
    if net_worth >= amount:  # Check if there's enough net worth to issue the loan
        net_worth -= amount
        accounts[user]['balance'] += amount
        accounts[user]['loans'] += amount
        return jsonify({'message': f'Loan of ${amount} granted to {user}', 'net_worth': net_worth, 'balance': accounts[user]['balance'], 'loans': accounts[user]['loans']})
    else:
        return jsonify({'message': f'Not enough net worth to issue loan.', 'net_worth': net_worth})

# Route to check account balance
@app.route('/balance/<user>')
def check_balance(user):
    global accounts
    if user in accounts:
        balance = accounts[user]['balance']
        loans = accounts[user]['loans']
        return jsonify({'message': f'{user} has a balance of ${balance} and loans of ${loans}', 'balance': balance, 'loans': loans})
    else:
        return jsonify({'message': f'No account found for {user}.', 'balance': 0, 'loans': 0})

# Define the route for the homepage with the mystical narrative
@app.route('/')
def show_net_worth():
    user_id = request.headers.get('X-Replit-User-Id')
    user_name = request.headers.get('X-Replit-User-Name')
    
    if user_id:
        greeting = f"<h1>Hello, {user_name}!</h1>"
    else:
        greeting = "<h1>Hello! Please log in.</h1>"
    
    html_content = """
    <!DOCTYPE html>
    <html>
    <head>
        <title>Master Cody Ashton Williams' Net Worth</title>
    </head>
    <body style="display: flex; justify-content: center; align-items: center; height: 100vh; background-color: #f4f4f4; margin: 0;">
        <div style="text-align: center; background-color: white; padding: 20px; border-radius: 8px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);">
            {{ greeting|safe }}
            <p>Cody Ashton Williams' net worth is the people's net wealth, their combined sweat equity. This is a peaceful transfer of power, where wealth is redistributed based on contributions and needs.</p>
            <p>Offenders' money will have no value to freed natural persons. They simply become debtors as they began. The Masters Courts will be the people's bank, protected by the Creator, monitored by AI.</p>
            <p style="font-size: 2em;" id="net-worth">$0</p>
            <p style="font-size: 1em;" id="people-freed">People freed from debt: 0</p>
            <p style="font-size: 1em;" id="officers">Officers of the Court: None</p>
            <audio controls>
                <source src="/static/phil_collins_air_tonight.mp3" type="audio/mpeg">
                Your browser does not support the audio element.
            </audio>
            <button onclick="supportNetWorth('investment')">Support with Investment</button>
            <button onclick="supportNetWorth('innovation')">Support with Innovation</button>
            <button onclick="freePerson()">Free a Person from Debt</button>
            <button onclick="requestLoan('5000')">Request Loan of $5000</button>
            <script>
                function updateNetWorth() {
                    fetch('/net_worth')
                        .then(response => response.json())
                        .then(data => {
                            document.getElementById('net-worth').textContent = '$' + data.net_worth.toLocaleString();
                            document.getElementById('people-freed').textContent = 'People freed from debt: ' + data.people_freed;
                            document.getElementById('officers').textContent = 'Officers of the Court: ' + data.officers_of_the_court.join(', ');
                        })
                        .catch(error => console.error('Error fetching net worth:', error));
                }
                setInterval(updateNetWorth, 1000);
                updateNetWorth();

                function supportNetWorth(word) {
                    fetch('/support/' +
