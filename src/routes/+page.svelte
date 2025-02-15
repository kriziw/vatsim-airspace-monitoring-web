<script>
	import "../lib/styles/index.css";
</script>

<svelte:head>
	<title>VatNotif</title>
</svelte:head>

<main class="container">
	<section class="hero">
		<h1 class="hero__title">Get notified when your favorite controllers come online</h1>
		<p class="hero__subtitle">
			VatNotif is a VATSIM notification service that allows you to receive notifications when your favorite controllers come online.
		</p>
		<a href="/dashboard" class="button button--primary">Get Started</a>
	</section>

	<section class="features">
		<div class="feature">
			<img src="/cards/explore.svg" alt="Find" class="feature__icon" />
			<h2 class="feature__title">Find</h2>
			<p class="feature__description">Find airports and positions you are interested in!</p>
		</div>
		<div class="feature">
			<img src="/cards/compass.svg" alt="Track" class="feature__icon" />
			<h2 class="feature__title">Track</h2>
			<p class="feature__description">Get their callsign and add them to your tracking list!</p>
		</div>
		<div class="feature">
			<img src="/cards/notification.svg" alt="Notify" class="feature__icon" />
			<h2 class="feature__title">Notify</h2>
			<p class="feature__description">Receive a notification once they come online!</p>
		</div>
	</section>

	<section class="getting-started">
		<h2 class="section-title">Getting Started</h2>
		<p class="getting-started__description">
			To get started, please login with your VATSIM account. After that you can proceed to the <a href="/dashboard">Dashboard</a> and add some callsigns
			you'd like to receive notifications for. You can also add Discord webhooks to receive notifications on your discord servers.
		</p>
		<a href="/dashboard" class="button button--primary">Get Started</a>
	</section>

	<section class="faq">
		<h2 class="section-title">FAQ</h2>
		<div class="faq__question">
			<h3 class="faq__question-title">What is VatNotif?</h3>
			<p class="faq__question-answer">
				VatNotif is a VATSIM notification service that allows you to receive notifications when your favorite controllers come online.
			</p>
		</div>
		<div class="faq__question">
			<h3 class="faq__question-title">How does it work?</h3>
			<p class="faq__question-answer">
				VatNotif uses the VATSIM API to fetch the latest data on controllers and their positions. It then compares this data with your
				preferences and sends you a notification if a match is found.
			</p>
		</div>
		<div class="faq__question">
			<h3 class="faq__question-title">Is the service paid?</h3>
			<p class="faq__question-answer">
				No, VatNotif is completely free to use. If you'd like to support the project, you can donate to the developer using <a
					href="https://paypal.me/kristn1">PayPal</a
				>.
			</p>
		</div>
		<div class="faq__question">
			<h3 class="faq__question-title">Is this project open-source?</h3>
			<p class="faq__question-answer">Yes, the project is open-source and can be found on GitHub:</p>
			<ul class="faq__question-list">
				<li><a href="https://github.com/kristiankunc/VatNotif-web">VatNotif-web</a></li>
				<li><a href="https://github.com/kristiankunc/VatNotif-api">VatNotif-api</a></li>
			</ul>
		</div>
	</section>

	<section class="news">
		<h2 class="section-title">What's new?</h2>
		<div class="news__article">
			<h3>Push Notifications</h3>
			<time datetime="2021-07-24">24. 7. 2021</time>
			<p>
				Push notifications have been added, so you can now receive notifications on your phone or desktop even when you're not on the site.
				They are available on all modern browsers and devices.
			</p>
		</div>
		<div class="news__article">
			<h3>Notifications Updates</h3>
			<time datetime="2021-06-16">16. 6. 2021</time>
			<p>
				The Discord notification embed look has been tweaked a little and a notification for when a controller logs off has been added. You
				can now do not have to check if a controller is still online or not.
			</p>
		</div>
		<div class="news__article">
			<h3>New UI</h3>
			<time datetime="2023-06-13">13. 6. 2023</time>
			<p>
				The VatNotif UI has been completely updated including the dashboard. This update hopes to make the site more appealing and
				user-friendly to newcomers.
			</p>
		</div>
	</section>

	<section>
		<h2 class="section-title">Contact</h2>
		<p>
			If you have any questions or concerns, please do not hesitate to contact us at <a href="mailto:kristian@kristn.co.uk"
				>kristian@kristn.co.uk</a
			>
			<br />
			Or get other contact options at <a href="https://kristn.co.uk">kristn.co.uk</a>
		</p>
	</section>
	<section class="data-handling">
		<h2 class="section-title">Data Handling Policy</h2>
		<p class="data-handling__description">
			At VatNotif, we take data privacy seriously and are committed to protecting your personal information. As such, we have implemented the
			following data handling policy:
		</p>
		<ol class="data-handling__list">
			<li class="data-handling__item">
				<strong>Unique Identifier:</strong> We only use a unique identifier (CID) from VATSIM to identify you. We do not store any other personal
				information.
			</li>
			<li class="data-handling__item">
				<strong>No Tracking/3rd Party Cookies:</strong> We do not use any tracking or 3rd party cookies. The only cookies we use are session cookies
				which are necessary for the website to function properly.
			</li>
			<li class="data-handling__item">
				<strong>Watched Callsigns and Notification Preferences:</strong> In our database, we only store your watched callsigns and notification
				preferences such as your Discord webhook URL. This information is used to provide you with real-time updates on your selected callsigns.
			</li>
			<li class="data-handling__item">
				<strong>Data Deletion:</strong> If you wish to have your data deleted, please send an email to
				<a href="mailto:kristian@kristn.co.uk">kristian@kristn.co.uk</a> We will promptly delete all of your stored data upon receipt of your request.
			</li>
		</ol>
		<p class="data-handling__disclaimer">
			<i
				>We hope that this policy provides you with peace of mind knowing that we are committed to protecting your personal information. If
				you have any questions or concerns, please do not hesitate to contact us.</i
			>
		</p>
	</section>
</main>
