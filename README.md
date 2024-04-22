<a href="https://reporacers.com/" taarget="_blank">
  <img src="https://github.com/repo-racers/.github/blob/main/profile/repo-racers.svg" alt="Repo Raacers" width="600px"/>
</a>

This repository is included in our open-source Pro Support service which offers an efficient solution for managing popular GitHub Actions dependencies with ease:

🙌 forked from [FranciscoKloganB/secrets-to-environment-variables-action](https://github.com/FranciscoKloganB/secrets-to-environment-variables-action)

<details>

<summary>What is Open-Source Pro Support?</summary>

Open-Source Pro Support is a comprehensive service designed to streamline your workflow by providing:

- **Customized Forks:** We create public forks of popular GitHub Actions, ensuring you have access to the latest features and fixes.
  
- **Dedicated Technical Support:** Say goodbye to the hassle of managing multiple open-source dependencies. With our service, you have a single point of contact for all your support needs. Reach out to us on our [Discord](https://discord.com/channels/1229786735161118882/1229786735161118885) server, and our team of experts will be ready to assist you.
  
- **Priority Fixes:** Experience seamless issue resolution with our priority fix service. If you encounter any issues with our forks, we prioritize fixing them promptly to minimize disruptions to your workflow.
  
- **Community Contribution:** We believe in giving back to the open-source community. When we fix issues in our forks, we handle creating pull requests to the original authors, ensuring that the entire community benefits from the improvements.

</details>

<details>

<summary>How It Works</summary>


1. **Choose Our Fork:** Instead of referencing popular GitHub Actions repositories directly, simply reference this repository in your workflow.
   
2. **Enjoy Dedicated Support:** If you encounter any issues or need assistance, reach out to us on our [Discord](https://discord.com/channels/1229786735161118882/1229786735161118885) server. Our team will be happy to help you promptly.
   
3. **Benefit from Priority Fixes:** Experience seamless issue resolution with our priority fix service. We prioritize fixing issues in our forks to ensure smooth operation for your projects.
   
4. **Contribute to the Community:** Rest assured that when we fix issues in our forks, we contribute back to the original repositories, benefiting the entire open-source community.

*Not Your Thing?*

We don't want to get in between you and the community. If you want to handle forking and submitting a pull request yourself, that's awesome.

However, feel free to reach out to us on [Discord](https://discord.com/channels/1229786735161118882/1229786735161118885) anyway if you need any help and advice in doing so.

:heart: [open-source](https://opensource.org/)

</details>

---

# GitHub secrets to environment variables

This Action reads your repository's GitHub Secrets and exports them as environment variables make them available to Actions within your Workflows.

It is possible to control what secrets are imported and how they are exported as environment variables.

- Include or exclude secrets (CSV or Regex)
- Add prefix to all exported environment variables
- Add suffix to all exported environment variables
- Remove prefix from all exported environment variables
- Remove suffix from all exported environment variables
- Override already existing variables (default is true)

Original credit goes to [oNaiPs](https://github.com/oNaiPs/secrets-to-env-action).

## Usage

Add the following action to your workflow:

```yaml
- uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
  with:
    secrets: ${{ toJSON(secrets) }}
```

After running this action, subsequent actions will be able to access the repository GitHub secrets as environment variables. Eliminating the need to read Secrets one by one and associating them with the respective environment variable.!

_Note the `secrets` key. It is **mandatory** so the action can read and export the secrets._

### Simple

```yaml
steps:
- uses: actions/checkout@v3
- uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
  with:
    secrets: ${{ toJSON(secrets) }}
- run: echo "Value of MY_SECRET: $MY_SECRET"
```

### Exclude secret(s) from list of secrets (CSV or Regular Expression)

Exclude defined secret(s) from list of secrets (comma separated, supports regex).

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: franciscokloganb/secrets-to-env-action@v1
    with:
      exclude: DUMMY_.+
      secrets: ${{ toJSON(secrets) }}
    # DUMMY_* IS NOT SUPPORTED
```

### Include secret(s) from list of secrets (CSV or Regular Expression)

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
    with:
      include: MY_SECRET, MY_OTHER_SECRETS_*
      secrets: ${{ toJSON(secrets) }}
  - run: echo "Value $MY_SECRET"
```

To export secrets that start with a given string, you can use `include: PREFIX_.+` or `PREFIX_.*`.

NOTE: If specified secret does not exist, it is ignored.

### Prefixing and Suffixing

It is possible to add and remove prefixes and suffixes from all the secrets found by this action. Refer to [Processing Algorithm](#algorithm-pipeline)

#### Add a prefix to exported secrets

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
    with:
      add-prefix: PREFIX_
      secrets: ${{ toJSON(secrets) }}
  - run: echo "Value $PREFIX_MY_SECRET"
```

#### Add a suffix to exported secrets

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
    with:
      add-suffix: _SUFFIX
      secrets: ${{ toJSON(secrets) }}
  - run: echo "Value $MY_SECRET_SUFFIX"
```

#### Remove a prefix

Remove a prefix to all exported secrets, if present.

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
    with:
      remove-prefix: PREFIX_
      secrets: ${{ toJSON(secrets) }}
  - run: echo "Value $MY_SECRET"
```

#### Remove a suffix

Remove a prefix to all exported secrets, if present.

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
    with:
      remove-suffix: _SUFFIX
      secrets: ${{ toJSON(secrets) }}
  - run: echo "Value $MY_SECRET"
```

### Overrides already existing variables (default is **false**)

```yaml
env:
  MY_SECRET: DONT_OVERRIDE
steps:
- uses: actions/checkout@v3
- uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
  with:
    override: true
    secrets: ${{ toJSON(secrets) }}
- run: echo "Value of MY_SECRET: $MY_SECRET" # possibly another value
```

### Convert environment variables case

Converts all exported secrets case to `lower` or `upper`. Default is `upper`.

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: repo-racers/secrets-to-environment-variables-action@v0.0.1
    with:
      convert: lower
      secrets: ${{ toJSON(secrets) }}
  - run: echo "Value $my_secret"
```

## Contributing

Run `pnpm install`, apply desired changes. When you are done. Open a Pull Request and Request a Review. Before the code is pushed we will run `pnpm validate` to ensure the code is working as intended.

## Algorithm Pipeline

This can be seen as a pipeline of changes in the following order:

- Read all secrets
- Apply inclusion filter
- Apply exclusion filter
- For all Keys repeat
  - Clone original Secret key
  - Remove prefix from cloned key if present
  - Remove suffix from cloned key if present
  - Add prefix to cloned key
  - Add suffix to cloned key
  - Convert case if applicable
  - Set Key unless key already exist and override is false
  - Export cloned key as an environment variable with its associated value

  ---

> [!TIP]
> For support with this repo and many other open-source projects, visit us at https://reporacers.com/ and join us on  [Discord](https://discord.com/channels/1229786735161118882/1229786735161118885).
