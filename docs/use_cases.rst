======================
 Use Cases & Examples
======================


Generate Session Token (default IAM User)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  ``Default`` profile in local awscli config. Default user has permissions to assume roles for which **stslib**
   will generate credentials
-  Token with default lifetime (60 minutes)
-  Cli *not* protected with MFA (Multi-Factor Authentication, 6 digit code)

.. code:: python


        from stslib import StsCore

        >>> sts_object = StsCore()
        >>> token = sts_object.generate_session_token()
        >>> print(token)
        <stslib.vault.STSToken at 0x7f05365e3ef0>

        # token attributes

        >>> print(token.start)
        datetime.datetime(2017, 8, 25, 20, 4, 37, tzinfo=tzutc()

        >>> print(token.end)
        datetime.datetime(2017, 8, 25, 21, 4, 36, tzinfo=tzutc())

        >>> print(token.access_key)
        'ASIAI6QV2U3JJAYRHCJQ'

        >>> print(token.secret_key)
        'MdjPAkXTHl12k64LSjmgTWMsmnHk4cJfeMHdXMLA'

        >>> print(token.session)
        'FQoDYXdzEDMaDHAaP2wi/+77fNJJryKvAa20AqGxoQlcRtf8RFLa5Mps9zK9V5SM3Q7+M3h9iNbcxfaZsUnTzFvFwjVZjYKk...zQU='

        >>> print(token.boto)    # native boto generated format

    {
        'AccessKeyId': 'ASIAI6QV2U3JJAYRHCJQ',
        'StartTime': datetime.datetime(2017, 8, 25, 20, 4, 37, tzinfo=tzutc()),
        'Expiration': datetime.datetime(2017, 8, 25, 21, 4, 36, tzinfo=tzutc()),
        'SecretAccessKey': 'MdjPAkXTHl12k64LSjmgTWMsmnHk4cJfeMHdXMLA',
        'SessionToken': 'FQoDYXdzEDMaDHAaP2wi/+77fNJJryKvAa20AqGxoQlcRtf8RFLa5Mps9zK9V5SM3Q7+M3h9iNbcxfa...zQU='
    }

---------------

Generate Session Token (named IAM User)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Named IAM user profile in local awscli config. User has permissions
   to assume roles for which **stslib**
   will generate credentials
-  MFA protected cli access configuration
-  STS Token with default lifetime (60 minutes)

.. code:: python


        from stslib import StsCore

        >>> sts_object = StsCore(profile_name='BobSmith')
        >>> code = '123456'
        >>> token = sts_object.generate_session_token(mfa_code=code)

        >>> print(token.boto)

    {
        'AccessKeyId': 'ASIAI6QV2U3JJAYRHCJQ',
        'StartTime': datetime.datetime(2017, 8, 25, 20, 4, 37, tzinfo=tzutc()),
        'Expiration': datetime.datetime(2017, 8, 25, 21, 4, 36, tzinfo=tzutc()),
        'SecretAccessKey': 'MdjPAkXTHl12k64LSjmgTWMsmnHk4cJfeMHdXMLA',
        'SessionToken': 'FQoDYXdzEDMaDHAaP2wi/+77fNJJryKvAdVZjYKk...zQU='
    }

---------------

Generate Credentials (1 hour lifetime)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  generate STS temporary credentials, default lifetime (60 minutes)
-  Credential format set to 'vault' (default stslib format)
-  **stslib** supports 2 credential formats. See the `Credential Format
   Overview <./primer/credential-format-overview.html>`__.

.. code:: python


        >>> sts_object = StsCore(profile_name='BobSmith')
        >>> token = sts_object.generate_session_token()
        >>> profile_list = [
                'DynamoDBRole-dev', 'CodeDeployRole-qa', 'S3ReadOnlyRole-prod'
            ]

                # where profile_list = list of profile names from local awscli config

        >>> sts_object.generate_credentials(profile_list)

        >>> print(credentials)

    {
        'sts-DynamoDBRole-dev': <stslib.vault.STSingleSet at 0x7fee0ae05c88>,
        'sts-CodeDeployRole-qa': <stslib.vault.STSingleSet at 0x7fee0ae05f60>,
        'sts-S3ReadOnlyRole-prod': <stslib.vault.STSingleSet at 0x7fee0ae05fd0>
    }

---------------

Generate Extended Use Credentials (Multi-hour, Auto-refresh)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Named IAM user profile in local awscli config. User has permissions
   to assume roles for which stslib
   will generate credentials
-  MFA protected cli configuration
-  Credential format set to 'boto' (native Amazon STS format)
-  Credentials auto-refreshed for total 5 hour valid lifetime without MFA auth

.. code:: python

        from stslib import StsCore

        >>> sts_object = StsCore(profile_name='BobSmith', format='boto')            # boto format credentials
        >>> code = '123456'
        >>> token = sts_object.generate_session_token(lifetime=5, mfa_code=code)    # 5 hour lifetime triggers auto-refresh
        >>> profile_list = [
                'DynamoDBRole-dev', 'CodeDeployRole-qa', 'S3ReadOnlyRole-prod'
            ]

                # where profile_list = list of profile names from local awscli config

        >>> sts_object.generate_credentials(profile_list)
        >>> credentials = sts_object.current_credentials


-  **Auto-Refresh of Credentials**: ``stslib`` will automatically generate new temporary credentials
   once per hour, prior to expiration (process below)

.. code:: python


        >>> print(credentials())

    {
      'sts-DynamoDBRole-dev': {
          'StartTime': datetime.datetime(2017, 10, 1, 14, 17, 45, 652218, tzinfo=<UTC>)},
          'Expiration': datetime.datetime(2017, 10, 1, 15, 17, 45, tzinfo=tzutc()),
          'AccessKeyId': 'ASIAJRW7F2BAVN4J34LQ',
          'SecretAccessKey': 'P8EjwTUKL4hil4Y7Ouo9OkFzQ1IxGikbhIjMP5uN',
          'SessionToken': 'FQoDYXdzEDMaDCpxZzDdwWGok/ylQiLcAdlrHCkxP+kvQOes3mnQ0r5GXt...'
      },
      'sts-CodeDeployRole-qa': {
          'StartTime': datetime.datetime(2017, 10, 1, 14, 17, 45, 652218, tzinfo=<UTC>)},
          'Expiration': datetime.datetime(2017, 10, 1, 15, 17, 45, tzinfo=tzutc()),
          'AccessKeyId': 'ASIAIOOOKUYFICAPC6TQ',
          'SecretAccessKey': '3Q+N4UMpbmW7OrvY2mfgbjXxr/qt1L4XqmO+Njpq',
          'SessionToken': 'FQoDYXdzEDMaDL/sJkeAF28UsxE/iyLUAbvBrCUoAkP/eqeS...'
      },
      'sts-S3ReadOnlyRole-prod': {
          'StartTime': datetime.datetime(2017, 10, 1, 14, 17, 45, 652218, tzinfo=<UTC>)}}
          'Expiration': datetime.datetime(2017, 10, 1, 15, 17, 46, tzinfo=tzutc()),
          'AccessKeyId': 'ASIAJPRTS4IXPYGPLKZA',
          'SecretAccessKey': 'EMAfJUz5zMNOyjKl7U2IWpJ0GCtWCos0squOE0wz',
          'SessionToken': 'FQoDYXdzEDMaDO0ekTXJi4+IRWV1ESLXAe1ZfOpmGcS9hbIr...'
      }
    }

    # stdout log stream
    /stslib/core.py - 0.2.0 - [INFO]: _validate: Valid account profile names: ['DynamoDBRole-dev', 'CodeDeployRole-qa', 'S3ReadOnlyRole-prod']
    /stslib/async.py - 0.2.0 - [INFO]: executing event: <bound method StsCore.generate_credentials of <stslib.core.StsCore object at 0x7f91c9df02e8>>
    /stslib/async.py - 0.2.0 - [INFO]: thread identifier: Thread-150
    /stslib/async.py - 0.2.0 - [INFO]: thread Alive status is: True
    /stslib/async.py - 0.2.0 - [INFO]: completed 1 out of 5 total executions
    /stslib/async.py - 0.2.0 - [INFO]: remaining in cycle: 4 hours, 59 minutes


      >>> print(credentials())

    {
      'sts-DynamoDBRole-dev': {
          'StartTime': datetime.datetime(2017, 10, 1, 15, 17, 45, 652218, tzinfo=<UTC>)},
          'Expiration': datetime.datetime(2017, 10, 1, 16, 17, 45, tzinfo=tzutc()),
          'AccessKeyId': 'ASIAJRW7F2BAVN4J34LQ',
          'SecretAccessKey': 'P8EjwTUKL4hil4Y7Ouo9OkFzQ1IxGikbhIjMP5uN',
          'SessionToken': 'FQoDYXdzEDMaDCpxZzDdwWGok/ylQiLcAdlrHCkxP+kvQOes3mnQ0r5GXt...'
      },
      'sts-CodeDeployRole-qa': {
          'StartTime': datetime.datetime(2017, 10, 1, 15, 17, 45, 652218, tzinfo=<UTC>)},
          'Expiration': datetime.datetime(2017, 10, 1, 16, 17, 45, tzinfo=tzutc()),
          'AccessKeyId': 'ASIAIOOOKUYFICAPC6TQ',
          'SecretAccessKey': '3Q+N4UMpbmW7OrvY2mfgbjXxr/qt1L4XqmO+Njpq',
          'SessionToken': 'FQoDYXdzEDMaDL/sJkeAF28UsxE/iyLUAbvBrCUoAkP/eqeS...'
      },
      'sts-S3ReadOnlyRole-prod': {
          'StartTime': datetime.datetime(2017, 10, 1, 15, 17, 45, 652218, tzinfo=<UTC>)}}
          'Expiration': datetime.datetime(2017, 10, 1, 16, 17, 46, tzinfo=tzutc()),
          'AccessKeyId': 'ASIAJPRTS4IXPYGPLKZA',
          'SecretAccessKey': 'EMAfJUz5zMNOyjKl7U2IWpJ0GCtWCos0squOE0wz',
          'SessionToken': 'FQoDYXdzEDMaDO0ekTXJi4+IRWV1ESLXAe1ZfOpmGcS9hbIr...'
      }
    }

    # stdout log stream
    /stslib/core.py - 0.2.0 - [INFO]: _validate: Valid account profile names: ['DynamoDBRole-dev', 'CodeDeployRole-qa', 'S3ReadOnlyRole-prod']
    /stslib/async.py - 0.2.0 - [INFO]: thread identifier: Thread-150
    /stslib/async.py - 0.2.0 - [INFO]: thread Alive status is: True
    /stslib/async.py - 0.2.0 - [INFO]: completed 2 out of 5 total executions
    /stslib/async.py - 0.2.0 - [INFO]: remaining in cycle: 3 hours, 59 minutes

Auto-Refresh Credentials -- Additional Info
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  Refresh of credentials is non-blocking (via threading)
-  Thread management is via event states; threads are terminated as soon as their associated session token expires or they receive a halt event.
-  No hanging threads. Any live threads when new credentials generated are safely terminated
   before generating a new set.

---------------

Non-default IAM Role credentials filename or location
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

| **Use-Case**: When you wish to use role credentials file not currently part of the awscli, provide a custom location to stslib as a parameter.

-  Initialization

.. code:: python

        import stslib

        >>> sts_object = stslib.StsCore()
        >>> credentials_file = '~/myAccount/role_credentials'   # awscli credentials file,
                                                                # located in ~/.aws
        >>> sts_object.refactor(credentials_file)
        >>> sts_object.profiles

-  Output

.. code:: json

    {
        "acme-db-dev": {
            "role_arn": "arn:aws:iam::236600111358:role/AcmeDEV",
            "mfa_serial": "arn:aws:iam::3788881165911:mfa/BillCaster",
            "source_profile": "william-caster"
        },
        "acme-apps-dev": {
            "role_arn": "arn:aws:iam::123660943358:role/AcmeDEV",
            "mfa_serial": "arn:aws:iam::3788881165911:mfa/BillCaster",
            "source_profile": "william-caster"
        },
        "acme-apps-qa": {
            "role_arn": "arn:aws:iam::430864833800:role/AcmeAdmin",
            "mfa_serial": "arn:aws:iam::3788881165911:mfa/BillCaster",
            "source_profile": "william-caster"
        },
        "acme-prod08": {
            "role_arn": "arn:aws:iam::798623437252:role/EC2RORole",
            "mfa_serial": "arn:aws:iam::3788881165911:mfa/BillCaster",
            "source_profile": "william-caster"
        },
        "acme-prod09": {
            "role_arn": "arn:aws:iam::123660943358:role/S3Role",
            "mfa_serial": "arn:aws:iam::3788881165911:mfa/BillCaster",
            "source_profile": "william-caster"
        }
    }


--------------

( `Table Of Contents <./index.html>`__ )

-----------------

|
