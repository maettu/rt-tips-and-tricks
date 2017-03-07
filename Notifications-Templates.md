Notes on Templates Used in Notifications
==========

A notification is an email you get when someone leaves a comment on a ticket or alike.

You are coding on the RT UI, in the forms you find, there.

The templates for this are under Admin > Global > Templates. To learn with template gets used where, look at column "Used by scrips" and compare this to Admin > Scrips

There are two kind of templates, "Perl" and "Simple". The following notes are on Perl templates.

See
    https://docs.bestpractical.com/rt/4.4.1/customizing/templates.html
    https://metacpan.org/pod/Text::Template

While mixing Perl code in your templates, be aware that every curly block scopes.

    { my $x = 5; }
    { $x; # undefined }

    { $x = 5; }
    { $x; # $x is 5 }

The last statement in a block is evaluated in scalar context and included in the resulting mail.

If you need a code block that does not spit out anything into the mail, use:

    {
        $x = 5;
        '';
    }

In the RT mail templates, everything before the first empty newline goes to the mail header, everything after goes to the mail body.
In consequence, you must not write code blocks that emit nothing (->empty line..) in the header.
If you need code in the header, write everything in one line (multiple statements are ok.) and make sure you print "Subject: " or alike on that same line.
That can make for very long lines.

Example: a template for "Admin Comment in HTML" for template 13 that sends notifications on comments in the language of the current owner of a ticket. (This is a working, but ugly solution, imo.)

    Subject: {$user = RT::User->new(); $user->Load($Ticket->Owner); $user_lang = $user->Lang; %comment = (de => 'Kommentar', fr => 'Commentaire', it => 'Commento', default => 'Comment');'';}[{$comment{$user_lang} // $somment{default}}] {my $s=($Transaction->Subject||$Ticket->Subject||""); $s =~ s/\[Comment\]\s*//g; $s =~ s/^Re:\s*//i; $s;}
    RT-Attach-Message: yes
    Content-Type: text/html

    {
        $ticket_id = $Ticket->id;
        $ticket_link = RT->Config->Get("WebURL").'Ticket/Display.html?id='.$ticket_id;
        $ticket_owner = $Ticket->Owner;

        $text_default =
        qq{<p>This is a comment about <a href="$ticket_link">ticket $ticket_id</a>. It is not sent to the Requestor(s):</p>
        <p>owner: $ticket_owner</p>
        <p>user language: $user_lang</p>};

        $text_de =
        qq{<p>Dies ist ein Kommentar zu <a href="$ticket_link">Ticket $ticket_id</a>. Er wurde nicht an die Ersteller des Tickets geschickt.</p>
        <p>Sprache des Besitzers: $user_lang</p>};

        # $text_fr and $text_it, here
        $text_fr =
        qq{TODO};

        $text_it =
        qq{TODO};

        %text = (
            default => $text_default,
            de => $text_de,
            fr => $text_fr,
            it => $text_it,
        );

        my $text = $text{default};
        $text = $text{$user_lang} if (exists $text{$user_lang});
        $text;
    }

    {$Transaction->Content(Type => "text/html")}

Another approach for the same problem would be to include everything in one code block and emit the mail at its end:

    {
    my $ticket_id = $Ticket->id;
    my $ticket_link = RT->Config->Get("WebURL").'Ticket/Display.html?id='.$ticket_id;
    my $ticket_owner = $Ticket->Owner;

    my $user = RT::User->new();
    $user->Load($Ticket->Owner);
    $user_lang = $user->Lang;

    # --------

    my $text_default =
    qq{<p>This is a comment about <a href="$ticket_link">ticket $ticket_id</a>. It is not sent to the Requestor(s):</p>
    <p>owner: $ticket_owner</p>
    <p>user language: $user_lang</p>};

    my $text_de =
    qq{<p>Dies ist ein Kommentar zu <a href="$ticket_link">Ticket $ticket_id</a>. Er wurde nicht an die Ersteller des Tickets geschickt.</p>
    <p>Sprache des Besitzers: $user_lang</p>};

    # $text_fr and $text_it, here
    my $text_fr =
    qq{on est là

    };

    my $text_it =
    qq{TODO};

    # ------


    my %comment = (
        de => 'Kommentar',
        fr => 'Commentaire',
        it => 'Commento',
        default => 'Comment'
    );

    %text = (
        default => $text_default,
        de => $text_de,
        fr => $text_fr,
        it => $text_it,
    );

    my $text = $text{default};
    $text = $text{$user_lang} if (exists $text{$user_lang});

    my $s=($Transaction->Subject||$Ticket->Subject||"");
    $s =~ s/\[Comment\]\s*//g;
    $s =~ s/^Re:\s*//i;

    my $subject = "Subject: [". ($comment{$user_lang} // $somment{default}) . "] $s :-)";

    my $header = "$subject
    RT-Attach-Message: yes
    Content-Type: text/html";

    my $mail = "$header\n\n$text";

    # "write" the whole mail in one.
    $mail;
    }

    {$Transaction->Content(Type => "text/html")}

This has the advantage of one code block / scope, no unwieldy lines.

Still better is an approach that exposes a function exposed from the backend, allowing to pass in only the texts from the UI.

    TODO
