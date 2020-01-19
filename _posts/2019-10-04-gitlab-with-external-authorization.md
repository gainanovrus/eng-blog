---
published: true
layout: single
title: GitLab CE in Docker. External OAuth2 Authorization
excerpt: >-
  GitLab allows to sign up and login with many external application like
  Google, Facebook, Github. Also a custom OAuth provider supported by
  a internal library - OmniAuth.
categories: devops
tags: docker gitlab devops
toc: true
# header:
#  teaser: /assets/images/custom-oauth2-provider-to-nextcloud.png
#  og_image: /assets/images/custom-oauth2-provider-to-nextcloud.png
last_modified_at: 2019-10-04
---

In the previous post was showed how to install GitLab CE.
I want to continue the work with GitLab and want to show how to set up some
configuration.

If you already has a system that stored a user profiles.
And the system provides an OAuth2 interface. You can use it as single sign-on
([SSO](sso)) point for other systems like GitLab.
You shouldn't create new users, not require to import any other profiles.

## Environment

1. GitLab CE 12.3.0 (Omnibus package for Docker) - `gitlab.example.com`
  - omniauth-oauth2-generic plugin (usually installed default)
2. External OAuth2 provider - `provider.example.com`

## Configuration

Full docs are available [here](https://docs.gitlab.com/ee/integration/omniauth.html).

Although the [official note](https://docs.gitlab.com/ee/integration/omniauth.html#using-custom-omniauth-providers)
the installed GitLab has already have the `omniauth-oauth2-generic` package.

1. Adding new application to our OAuth2 provider
and input Redirect URI to our GitLab:
```
Redirect URI: https://gitlab.example.com/users/auth/<provider>/callback
```
Where is `<provider>` is name of our External OAuth2 provider (i.e. `provider.example.com`) .

2. After submit a creation, you will get `app-id` and `app-secret`. Keep in mind them.

3. Open GitLab configuration file - `gitlab.rb`. Find `OmniAuth Settings` section.

Add next settings to the file:
```ruby
gitlab_rails['omniauth_enabled'] = true
gitlab_rails['omniauth_allow_single_sign_on'] = true
gitlab_rails['omniauth_block_auto_created_users'] = false
gitlab_rails['omniauth_providers'] = [
  {
    'name' => 'oauth2_generic',
    'app_id' => 'YOUR-APP-ID',
    'app_secret' => 'YOUR-APP-SECRET',
    'args' => {
      client_options: {
        'site' => 'https://provider.example.com', # including port if necessary
        'user_info_url' => '/api/v1/session',     # user profile in JSON
        'authorize_url' => '/oauth/authorize',    # The authorization endpoint for your OAuth server
        'token_url' => '/oauth/token'             # The token request endpoint for your OAuth server
      },
      user_response_structure: {         # change this if necessary (there is my configuration)
        root_path: ['current_user'],     # i.e. if attributes are returned in JsonAPI format (in a 'user' node nested under a 'data' node)
        attributes: { nickname: 'name' } # if the nickname attribute of a user is called 'username'
      },
      # optionally, you can add the following two lines to "white label" the display name
      # of this strategy (appears in urls and Gitlab login buttons)
      # If you do this, you must also replace oauth2_generic, everywhere it appears above, with the new name.
      name: 'Provider', # display name for this strategy
      strategy_class: "OmniAuth::Strategies::OAuth2Generic" # Devise-specific config option Gitlab uses to find renamed strategy
    }
  }
]
```

Change `YOUR-APP-ID` and `YOUR-APP-SECRET` to the generated `app-id` and `app-secret` earlier.
And check the `client_options` and `user_response_structure`.
I've change them to my provider.
Save the modification `gitlab.rb` file.

4. Restart GitLab. Run command in the console on the host

```
# gitlab-ctl reconfigure
```

5. Optional. Disable standard sign up and sign in option in the first screen.

Login as admin and following to the section: **Admin -> Settings -> General** -
https://gitlab.example.com/admin/application_settings

![signup-disable]({{ site.url }}{{ site.baseurl }}/assets/images/gitlab-docker/signup-disable.png){: .align-center}
![signin-disable]({{ site.url }}{{ site.baseurl }}/assets/images/gitlab-docker/signin-disable.png){: .align-center}

This keeps only authorize with our configured OAuth2 Provider.

That's all. Open another browser (or incognito mode of current one) and test the
authorization process with external sign up.

After successful login return to the browser with admin session and set up
the new user that authorized before as a new admin.

## Some notes

If you want to enable sign-in after you disable sign-in.
There is no reconfigure option in omnibus-gitlab that will enable that again.
After `gitlab-rails console` opens up,
type `ApplicationSetting.last.update_attributes(signin_enabled: true)`
and then `gitlab-ctl restart`.
The solution was found [here](https://gitlab.com/gitlab-org/omnibus-gitlab/issues/561).
I found it after I block the internal authorization and can't login as admin.

## Additional information

* [omniauth-oauth2-generic gem](https://gitlab.com/satorix/omniauth-oauth2-generic) -
    Provides an OmniAuth strategy for authenticating with an External OAuth2 service.
* [OmniAuth](https://docs.gitlab.com/ee/integration/omniauth.html) -
    Allows users to sign in using Twitter, GitHub, and other popular services.
* [Enable signin repeat](https://gitlab.com/gitlab-org/omnibus-gitlab/issues/561) -
    How to enable signin after disable signin in the application setting.
* [GitLab Feature Comparison](https://about.gitlab.com/pricing/self-managed/feature-comparison/) -
    Find the difference between version of self-hosted GitLab.

[sso]: https://en.wikipedia.org/wiki/Single_sign-on
