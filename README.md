# Velxio CI GitHub Action

Run your firmware in the [Velxio](https://velxio.dev) simulator from a
GitHub Actions job: boot the real binary on a simulated board, check the
serial output, drive buttons and sensors with a scenario, take screenshots
of displays, and fail the job when the firmware misbehaves.

Velxio CI is a paid feature of the Maker and Pro plans (200 and 2,000
simulated minutes per month). Create a token at
<https://velxio.dev/account/ci> and store it as a repository secret.

## Usage

```yaml
- name: Test with Velxio
  uses: velxio/velxio-ci-action@v1
  with:
    token: ${{ secrets.VELXIO_CLI_TOKEN }}
    path: firmware/blink
    timeout: 10000
    expect_text: 'Hello, world!'
    fail_text: 'Error'
    scenario: 'test-hello-world.yaml'
    serial_log_file: 'log-hello-world.txt'
```

The project directory needs a `velxio.toml` (or a `wokwi.toml`) naming the
compiled firmware, and a `diagram.json` describing the circuit. Compile the
firmware in an earlier step with your own toolchain (arduino-cli, idf.py,
PlatformIO, cargo); Velxio only runs it.

## Inputs

| input | default | meaning |
|---|---|---|
| `token` | required | Velxio CI token |
| `path` | `.` | project directory (velxio.toml / wokwi.toml + diagram.json) |
| `timeout` | `10000` | simulated-time budget in ms |
| `expect_text` | | pass as soon as this appears on serial |
| `fail_text` | | fail as soon as this appears on serial |
| `scenario` | | automation scenario YAML |
| `serial_log_file` | | write the whole serial log here |
| `diagram_file` | `diagram.json` | circuit file |
| `elf` | | ELF firmware (overrides the toml) |
| `firmware` | | `.hex` / `.bin` / `.uf2` / merged ESP32 image (overrides the toml) |
| `screenshot_part`, `screenshot_time`, `screenshot_file` | | take a screenshot of a part at a simulated time |
| `timeout_exit_code` | `42` | exit code when the budget is reached (`0` to just collect serial for N ms) |
| `server` | `https://velxio.dev` | Velxio server |
| `cli_version` | `latest` | velxio-cli release tag |

## Outputs

`run_id`, `run_url`, `status` (`passed`, `failed`, `timeout`, `error`,
`cancelled`, `lost`) and `sim_time_ms`.

## Exit codes

The step fails when velxio-cli exits non-zero: `1` a check failed, `2` a
configuration or lint error, `3` an authentication error, `4` no CI minutes
left this month, `5` a server or runner error, `42` (or `timeout_exit_code`)
the simulated-time budget ran out. Full list in the
[velxio-cli docs](https://github.com/velxio/velxio-cli/blob/main/docs/exit-codes.md).

## Migrating from another simulator's CI action

The inputs use the same names on purpose. Change the `uses:` line, rename
the secret, and keep your `diagram.json` and scenario files: Velxio reads
`wokwi.toml` too. Boards, parts and scenario steps that Velxio does not
support are reported by name, never silently replaced.

## License

MIT. The action installs the MIT-licensed `velxio-cli` binary; the
simulation itself runs on Velxio's servers under your plan.
