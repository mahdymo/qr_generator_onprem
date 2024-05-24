# Locally owned QR Generator 

## Overview 

In this repo. we aim to create a self-obtained QR Code generator that replaces Google quickchart. while quickchart if an amazing way to kickstart testing and some projects, it worth noting that some organizations don't prefer to send over the secret keys and users' IDs over to 3rd parties without proper liability contract in place. 

* One question always rises why a lot of people use quickchart service if it's not that secure? 
Many rely on the fact that anonymity gives that sense of security, as quickchart doesn't keep track of to which secret key this organization belong to, then no one can track it back to us. I believe you can get how this is not totally true. 

## Prerequisites 

In order to have this app deployed, you need the below, 
- python3
- pip
- modules: flask, totp, qr_code 


## Installation 

1- Install prerequisites
   ```
   sudo apt update
   sudo apt install python3 python3-pip
   pip3 install qrcode pyotp flask
   ```
2- Create flask application (for web application interface)
- Create folder `mkdir qr_generator` 
- Access folder `cd qr_generator`
- Create app.py file 
```python
   from flask import Flask, request, send_file
   import qrcode
   import io
   import pyotp

   app = Flask(__name__)

   @app.route('/generate_totp_qr', methods=['GET'])
   def generate_totp_qr():
       # Get the user information from the request
       user = request.args.get('user')
       secret = request.args.get('secret')
       issuer = request.args.get('issuer')

       if not user or not secret or not issuer:
           return "Error: 'user', 'secret', and 'issuer' parameters are required.", 400

       # Create a TOTP object
       totp = pyotp.TOTP(secret)

       # Generate the provisioning URI
       uri = totp.provisioning_uri(user, issuer_name=issuer)

       # Generate the QR code
       qr = qrcode.QRCode(
           version=1,
           error_correction=qrcode.constants.ERROR_CORRECT_L,
           box_size=10,
           border=4,
       )
       qr.add_data(uri)
       qr.make(fit=True)

       img = qr.make_image(fill='black', back_color='white')

       # Create a bytes buffer to hold the image
       img_io = io.BytesIO()
       img.save(img_io, 'PNG')
       img_io.seek(0)

       # Send the image file
       return send_file(img_io, mimetype='image/png')

   if __name__ == '__main__':
       app.run(host='0.0.0.0', port=5000)
   ```
3- Create service to automatically run this app. 
`sudo nano /etc/systemd/system/flask_app.service`

- Add the following to the file. 
```
[Unit]
Description=Flask Application

[Service]
ExecStart=/usr/bin/python3 /path/to/your/flask_app/app.py
WorkingDirectory=/path/to/your/flask_app
Restart=always
User=your_username

[Install]
WantedBy=multi-user.target
```

- Replace `/path/to/your/flask_app` with the actual path. 
- Run `sudo systemctl daemon-reload`.
- Run `sudo systemctl start flask_app`.


4- Testing the Application Locallyز
   Open your web browser or use `curl` to test the application. For example, to generate a QR code, use the following URL structure:
   ```
   http://localhost:5000/generate_totp_qr?user=username&secret=your_secret_key&issuer=YourApp
   ```

   Replace `username`, `your_secret_key`, and `YourApp` with the appropriate values.

5- Example Usage with curl:
```bash
curl "http://localhost:5000/generate_totp_qr?user=username&secret=your_secret_key&issuer=YourApp" --output totp_qr_code.png
```
