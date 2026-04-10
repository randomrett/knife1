# knife1
this is a git repo.
import os
import requests
from flask import Flask, request, jsonify, send_from_directory, Response

app = Flask(__name__, static_folder='../../frontend', static_url_path='')

EVENT_SERVICE_URL = "http://localhost:5002"
REGISTRATION_SERVICE_URL = "http://localhost:5003"

@app.route('/')
def index():
    return send_from_directory(app.static_folder, 'index.html')

@app.route('/<path:path>')
def serve_static(path):
    return send_from_directory(app.static_folder, path)

# Proxy for Events
@app.route('/api/events', methods=['GET', 'POST'])
def proxy_events():
    try:
        url = f"{EVENT_SERVICE_URL}/events"
        if request.method == 'GET':
            resp = requests.get(url, timeout=5)
        else:
            resp = requests.post(url, json=request.json, timeout=5)
        return Response(resp.content, resp.status_code, resp.raw.headers.items())
    except requests.exceptions.RequestException as e:
        return jsonify({"error": "Event service is temporarily unavailable.", "details": str(e)}), 503

@app.route('/api/events/<int:event_id>', methods=['GET'])
def proxy_event_details(event_id):
    try:
        resp = requests.get(f"{EVENT_SERVICE_URL}/events/{event_id}", timeout=5)
        return Response(resp.content, resp.status_code, resp.raw.headers.items())
    except requests.exceptions.RequestException as e:
        return jsonify({"error": "Event service is temporarily unavailable."}), 503

# Proxy for Registration
@app.route('/api/register', methods=['POST'])
def proxy_register():
    try:
        resp = requests.post(f"{REGISTRATION_SERVICE_URL}/register", json=request.json, timeout=5)
        return Response(resp.content, resp.status_code, resp.raw.headers.items())
    except requests.exceptions.RequestException as e:
        return jsonify({"error": "Registration service is temporarily unavailable."}), 503

if __name__ == '__main__':
    print("Starting Gateway Service on port 5001...")
    # Serve on port 5001
    app.run(port=5001, debug=True, threaded=True)
