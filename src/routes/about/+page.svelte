<svelte:head>
	<title>About SSL and Our Tool</title>
	<meta name="description" content="About SSL and how our tool works" />
</svelte:head>

<div class="text-column">
	<h1>About SSL and This Tool</h1>

	<p class="mb-4">
		SSL (Secure Sockets Layer) is a security protocol that provides encrypted communication between a web server and a browser. It ensures that data exchanged between users and websites remains private and integral. The main goal is to prevent eavesdropping and tampering of data as it travels over the internet.
	</p>

	<p class="mb-4">
		SSL Certificates are digital certificates that validate the identity of a website and encrypt information sent to the server. When installed on a web server, an SSL Certificate activates a padlock icon and the HTTPS protocol, indicating secure connections from a web server to a browser.
	</p>

	<p class="mb-4">
		This tool is designed to help users verify and validate the SSL status of a website. On the backend, we utilize the OpenSSL command-line toolkit. OpenSSL is a robust, open-source toolkit that implements the Secure Sockets Layer (SSL) and Transport Layer Security (TLS) protocols. By leveraging OpenSSL, our tool can fetch, verify, and validate the SSL certificate of any given domain efficiently.
	</p>

	<p class="mb-4">
		The OpenSSL CLI (Command Line Interface) provides a set of commands that allows users to manipulate SSL and TLS related data. For example, the command <code>openssl x509 -in certificate.crt -text -noout</code> can be used to display the certificate information. Using these commands, our backend processes and validates SSL details efficiently.
	</p>

	<p>
		The primary purpose of this tool is to ensure that your website or any site you interact with maintains the highest standards of security. By regularly checking the SSL status, you can be confident in the safety of your data and online interactions.
	</p>
</div>
