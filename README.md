NewsApp Frontend Plugin
=======================

Introduction
------------
This is a Javascript library created to query a news server app and deliver news
feeds to a user running a remote application. The plugin is included as any 
Javascript file and overrides the main landing page of the application by providing
its own landing page which a user first interacts with before using the main system.

Some links on the landing page display more on specific news categories whereas 
a dedicated link directs the user to the main system. A banner also shows on every
page which includes the file to continuously supply available news headings.

How to Use
==========

1. Create glue logic on the server side for your application to reroute AJAX data 
		calls which can't easily be directed to the news server without violatiing 
		Cross-origin resource sharing (CORS) constraints. Only 2 connections are 
		necessary in the database. If using Node.js, example code for the connections
		is as follows:
		
			module.exports = function (router, session_username) {
					var url = require('url');
					var data_connection = {
						  "ip_address": "http://NEWS_APP_IP_ADDRESS:NEWS_APP_PORT",
						  "news_path": "/api/news_feed?ip_address=",
						  "logs_path": "/api/log",
						  "username": "USERNAME",
						  "password": "PASSWORD"
					}
					router.route('/')
						  .get(function (req, res) {
						      res.render('index', {});
						  });
					router.route('/query')
						  .get(function (req, res) {						      
						      var Client = require('node-rest-client').Client;
						      var ip_address = req.connection.remoteAddress;
						      (new Client()).get(data_connection.ip_address + data_connection.news_path + ip_address, function (data, response) {
						          var ip_address = req.connection.remoteAddress;
						          var news = JSON.parse(data);
						          news.ip_address = ip_address;
						          res.status(200).json(news);
						      });
						  });
					router.route('/log')
						  .get(function (req, res) {
						      var url_parts = url.parse(req.url, true);
						      var query = url_parts.query;
						      var Client = require('node-rest-client').Client;
						      (new Client()).get(data_connection.ip_address + data_connection.logs_path + "?news_id=" + query.news_id + "&category=" +
						          query.category + "&ip_address=" + query.ip_address, function (data, response) {
						          var result = JSON.parse(data);
						          res.status(200).json(result);
						      });
						  });
					return router;
			}
			
		The plugin expects 2 paths, "/query" and "/log?news_id=NEWS_ID&category=CATEGORY&ip_address=IP_ADDRESS"
		
2. Include the library on every page where news articles can be called from e.g. "<script src='/js/bannerNews.js'></script>"


