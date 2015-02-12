# WWW-Crawler-Mojo

WWW::Crawler::Mojo is a web crawling framework written in Perl on top of mojo toolkit, allowing you to write your own crawler rapidly. 

***This software is considered to be alpha quality and isn't recommended for regular usage.***

## Features

* Easy to rule your crawler.
* Allows to use [Mojo::URL] for URL manipulations, [Mojo::Message::Response] for response manipulation and [Mojo::DOM] for DOM inspection.
* Internally uses [Mojo::UserAgent] which supports non-blocking I/O HTTP and WebSocket with IPv6, TLS, SNI, IDNA, HTTP/SOCKS5 proxy, Comet (long polling), keep-alive, connection pooling, timeout, cookie, multipart, gzip compression.
* Throttle the connection with max connection and max connection per host options.
* Depth detection.
* Tracks 301 HTTP redirects.
* Network error detection.
* Retry with your own rules.
* Shuffles queue periodically if indicated.
* Peeping server for crawler development.
* Crawls beyond basic authentication.
* Form submitting emulation.
* Flexible enough to crawl even 404 error documents.

[Mojo::URL]:http://mojolicio.us/perldoc/Mojo/URL
[Mojo::DOM]:http://mojolicio.us/perldoc/Mojo/DOM
[Mojo::Message::Response]:http://mojolicio.us/perldoc/Mojo/Message/Response
[Mojo::UserAgent]:http://mojolicio.us/perldoc/Mojo/UserAgent

## Requirements

* Perl 5.14
* Mojolicious 5.75

## Synopsis

    use WWW::Crawler::Mojo;
    
    my $bot = WWW::Crawler::Mojo->new;
    
    $bot->on(res => sub {
        my ($bot, $scrape, $job, $res) = @_;
        
        $scrape->() if (...); # collect URLs from this document
    });
    
    $bot->on(refer => sub {
        my ($bot, $enqueue, $job, $context) = @_;
        
        $enqueue->() if (...); # enqueue this job
    });
    
    $bot->enqueue('http://example.com/');
    $bot->crawl;

## Installation

    $ cpanm WWW::Crawler::Mojo

## Documentation

* [WWW::Crawler::Mojo](http://search.cpan.org/perldoc?WWW%3A%3ACrawler%3A%3AMojo)
* [WWW::Crawler::Mojo::Job](http://search.cpan.org/perldoc?WWW%3A%3ACrawler%3A%3AMojo%3A%3AJob)
* [WWW::Crawler::Mojo::UserAgent](http://search.cpan.org/perldoc?WWW%3A%3ACrawler%3A%3AMojo%3A%3AUserAgent)

## Examples

Restrict enqueuing URLs by depth.

    $bot->on(refer => sub {
        my ($bot, $enqueue, $job, $context) = @_;
        
        $enqueue->() if ($job->depth < 5);
    });

Restrict enqueuing URLs by host.

    $bot->on(refer => sub {
        my ($bot, $enqueue, $job, $context) = @_;
        
        $enqueue->() if $job->resolved_uri->host eq 'example.com';
    });

Restrict enqueuing URLs by referrer's host.

    $bot->on(refer => sub {
        my ($bot, $enqueue, $job, $context) = @_;
        
        $enqueue->() if $job->referrer->resolved_uri->host eq 'example.com';
    });

Excepting enqueuing URLs by path.

    $bot->on(refer => sub {
        my ($bot, $enqueue, $job, $context) = @_;
        
        $enqueue->() unless ($job->resolved_uri->path =~ qr{^/foo/});
    });

Restricting following URLs by host on response event.

    $bot->on(res => sub {
        my ($bot, $scrape, $job, $res) = @_;
        
        $scrape->() if ($job->resolved_uri->host eq 'example.com');
    });

Speed up.

    $bot->max_conn(5);
    $bot->max_conn_per_host(5);

Authentication. The user agent automatically reuses the credential for the host.

    $bot->enqueue('http://jamadam:password@example.com');

Want to scrape inside error documents?

    $bot->on(res => sub {
        my ($bot, $scrape, $job, $res) = @_;
        
        $bot->scrape_any($res, $job); # doesn't restrict to 200 OK
    });

You can fulfill any prerequisites such as login form submittion so that a login session will be established with cookie or something.

    my $bot = WWW::Crawler::Mojo->new;
    $bot->ua->post('http://example.com/admin/login', form => {
        username => 'jamadam',
        password => 'password',
    });
    $bot->enqueue('http://example.com/admin/');
    $bot->crawl

By peeping server, you can peep into the crawler attributes such as queue during crawling.

    $bot->peeping_port(3030);
    
    # Following URL will be available
    # http://127.0.0.1:3030//dumper/queue

## Other examples

* [WWW-Flatten](https://github.com/jamadam/WWW-Flatten)
* See the scripts under the example directory.

## Copyright

Copyright (C) jamadam

This program is free software; you can redistribute it and/or
modify it under the same terms as Perl itself.

