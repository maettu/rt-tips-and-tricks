Database Related Stuff
======================
The collections like "Users.pm" derive from DBIx::SearchBuilder

There is no new(), DBIx::SearchBuilder calls
    _Init

To see the current query:

    my $QueryString = $self->BuildSelectQuery();

