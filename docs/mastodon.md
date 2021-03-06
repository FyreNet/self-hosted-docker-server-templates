# Upgrading Mastdon

If you are adding Mastodon to your stack, after you edit add-ons/docker-compose.Mastodon.yaml, updating the version of the mastodon images to one of the tags available on [this page](https://hub.docker.com/r/tootsuite/mastodon/tags/)(e.g. image: tootsuite/mastodon:v2.3.3), you must run the following commands to prepare your Mastodon instance:

```bash
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml up mastodon-db mastodon-redis
docker-compose run --rm mastodon-web bundle exec rake secret
#copy the output into configs/mastodon.env for the value of SECRET_KEY_BASE
docker-compose run --rm mastodon-web bundle exec rake secret
#copy the output into configs/mastodon.env for the value of OTP_SECRET
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake mastodon:webpush:generate_vapid_key
#copy the output into configs/mastodon.env for the values of VAPID_PRIVATE_KEY and VAPID_PUBLIC_KEY
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake db:migrate
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web chown -R 991:991 /mastodon/public/assets /mastodon/public/packs /mastodon/public/system
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake assets:precompile
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake mastodon:add_user
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake mastodon:confirm_email USER_EMAIL=your@email
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake mastodon:make_admin USERNAME=yourname
###after running all of the above commands, you can now bring your entire stack up with the up -d command.  Make sure to -f all of your compose files!
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml up -d
```

To update mastodon, change the version in the image: clause of mastodon-web, mastodon-streaming, and mastodon-sidekiq (e.g. image: tootsuite/mastodon:v2.3.3), then run the following commands:

```bash
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml stop mastodon-web mastodon-streaming mastodon-sidekiq
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake db:migrate
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml run --rm mastodon-web bundle exec rake assets:precompile
###after running all of the above commands, you can now bring your entire stack up with the up -d command.  Make sure to -f all of your compose files!
docker-compose -f docker-compose.base.yaml -f add-ons/docker-compose.Mastodon.yaml up -d
```

This updates the database to be compatible with the new version and builds any new assets.

For more rake commands, including other administrative commands, refer to [this guide.](https://github.com/tootsuite/documentation/blob/master/Running-Mastodon/List-of-Rake-tasks.md)
