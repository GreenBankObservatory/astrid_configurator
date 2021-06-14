<script lang="ts">
  import { createForm } from "felte";
  import reporter from "@felte/reporter-dom";
  import { validator } from "@felte/validator-yup";
  import * as yup from "yup";
  const schema = yup.object({
    conf_name: yup.string().required(),
    rx: yup.string().ensure().required(),
    nwin: yup.number().typeError("Must be a positive number").positive(),
    rest_freq: yup.number().typeError("Must be a positive number").positive(),
    bandwidth: yup.number().typeError("Must be a positive number").positive(),
    nchan: yup
      .number()
      .typeError("Must be a positive integer")
      .positive()
      .integer(),
    tint: yup.number().typeError("Must be a positive number").positive(),
    vframe: yup.string().required(),
    framevdef: yup.string().required(),
    use_cal: yup.boolean().required(),
    noisecal: yup.string(),
    pol: yup.string().required(),
    xcor: yup.boolean().typeError("Required").required(),
  });
  const receivers = [
    { value: "Rcvr_340", label: "P-band" },
    { value: "Rcvr_800", label: "800 MHz" },
    { value: "Rcvr_1070", label: "PF2_1070" },
    { value: "Rcvr1_2", label: "L-band" },
    { value: "Rcvr2_3", label: "S-band" },
    { value: "Rcvr4_6", label: "C-band" },
    { value: "Rcvr8_10", label: "X-band" },
    { value: "Rcvr12_18", label: "Ku" },
    { value: "RcvrArray18_26", label: "KFPA" },
    { value: "Rcvr26_40", label: "Ka" },
    { value: "Rcvr40_52", label: "Q-band" },
    { value: "Rcvr68_92", label: "W-band" },
  ];
  // mode, bandwidth, # channels
  const vegasModes = [
    [1, 1500, 1024],
    [2, 1500, 16384],
    [3, 1080, 16385],
    [4, 187.5, 32768],
    [5, 187.5, 65536],
    [6, 187.5, 131072],
    [7, 100, 32768],
    [8, 100, 65536],
    [9, 100, 131072],
    [10, 23.44, 32768],
    [11, 23.44, 65536],
    [12, 23.44, 131072],
    [13, 23.44, 262144],
    [14, 23.44, 524288],
    [15, 11.72, 32768],
    [16, 11.72, 65536],
    [17, 11.72, 131072],
    [18, 11.72, 262144],
    [19, 11.72, 524288],
  ];
  const bandwidths = [
    { value: 1500, label: "1500 MHz" },
    { value: 1080, label: "1080 MHz" },
    { value: 187.5, label: "187.5 MHz" },
    { value: 100, label: "100 MHz" },
    { value: 23.44, label: "23.44 MHz" },
    { value: 11.72, label: "11.72 MHz" },
  ];

  let channels = [];
  $: {
    channels = vegasModes
      .filter(([mode, bw, nchan]) => Number(bw) === Number($data.bandwidth))
      .map(([mode, bw, nchan]) => ({
        value: nchan,
        label: nchan,
      }));
    channels.length && setField("nchan", channels[0].value);
  }
  const { form, isValid, data, errors, setField, setFields, reset } =
    createForm({
      onSubmit: (data) => console.log("Submit", data),
      onError: (error) => error,
      extend: [validator, reporter({ single: true })],
      validateSchema: schema,
      initialValues: {
        conf_name: "",
        rx: "",
        nwin: null,
        rest_freq: null,
        bandwidth: null,
        nchan: null,
        tint: null,
        vframe: "",
        framevdef: "",
        use_cal: false,
        noisecal: "",
        pol: "",
        xcor: false,
      },
    });
  function genDummyData() {
    const _dummyData = {
      conf_name: "dummy inputs",
      rx: "Rcvr_340",
      nwin: 1,
      rest_freq: 1,
      bandwidth: 1080,
      nchan: 16385,
      tint: 1,
      vframe: "lsrk",
      framevdef: "radio",
      use_cal: false,
      noisecal: "high",
      pol: "linear",
      xcor: true,
    };
    setFields(_dummyData);
    // TODO: Is there a away to avoid doing this manually?
    document.getElementById("rx-select").value = _dummyData.rx;
    document.getElementById("bandwidth-select").value = _dummyData.bandwidth;
    document.getElementById("nchan-select").value = _dummyData.nchan;
  }
  function genConfigString(data) {
    return `${data.conf_name.replace(" ", "_")}="""
  receiver="${data.rx}",
  obstype="Spectroscopy",
  backend="VEGAS",
  nwin=${data.nwin},
  restfreq=${data.rest_freq},
  deltafreq=0.0,
  bandwidth=${data.bandwidth},
  nchan=${data.nchan},
  swmode="tp",
  swtype=None ,
  swper=1.0,
  swfreq="0,0",
  tint=${data.tint},
  vframe="${data.vframe}",
  framevdef="${data.framevdef}",
  noisecal="${data.noisecal}",
  pol="${data.pol}",
  xcor=${data.xcor ? "True" : "False"},
  notchfilter="In",
"""`;
  }
</script>

<main class="container">
  <h1>Generate AstrID Config</h1>
  <h2>Position Switched ON/OFF Observation</h2>
  <div class="row">
    <div class="col-lg-6">
      <form use:form>
        <div class="row mb-1">
          <label class="col-sm-6 col-form-label" for="conf_name"
            >Name of the Config section</label
          >
          <div class="col-sm-6">
            <input class="form-control" type="text" name="conf_name" />
            <div
              class="error-msg"
              id="conf_name-validation"
              data-felte-reporter-dom-for="conf_name"
            />
          </div>
        </div>

        <div class="row mb-1">
          <label class="col-form-label col-sm-6 pt-0" for="rx">Receiver</label>
          <div class="col-sm-6">
            <select
              class="form-select"
              id="rx-select"
              name="rx"
              aria-label="Select receiver"
            >
              <option value="">Select Receiver</option>
              {#each receivers as receiver}
                <option value={receiver.value}>{receiver.label}</option>
              {/each}
            </select>
            <div
              class="error-msg"
              id="rx-validation"
              data-felte-reporter-dom-for="rx"
            />
          </div>
        </div>

        <div class="row mb-1">
          <label class="col-sm-6 col-form-label" for="nwin"
            ># of spectral windows</label
          >
          <div class="col-sm-6">
            <input
              class="form-control"
              type="text"
              inputmode="numeric"
              name="nwin"
            />
            <div
              class="error-msg"
              id="nwin-validation"
              data-felte-reporter-dom-for="nwin"
            />
          </div>
        </div>

        <div class="row mb-1">
          <label class="col-sm-6 col-form-label" for="rest_freq"
            >Rest Frequency</label
          >
          <div class="col-sm-6">
            <input
              class="form-control"
              type="text"
              inputmode="numeric"
              name="rest_freq"
            />
            <div
              class="error-msg"
              id="rest_freq-validation"
              data-felte-reporter-dom-for="rest_freq"
            />
          </div>
        </div>

        <div class="row mb-1">
          <label class="col-form-label col-sm-6 pt-0" for="bandwidth"
            >Bandwidth (MHz)</label
          >
          <div class="col-sm-6">
            <select
              class="form-select"
              id="bandwidth-select"
              name="bandwidth"
              aria-label="Select number of bandwidths"
            >
              <option value="">Select Bandwidth</option>
              {#each bandwidths as bandwidth}
                <option value={bandwidth.value}>{bandwidth.label}</option>
              {/each}
            </select>
            <div
              class="error-msg"
              id="bandwidth-validation"
              data-felte-reporter-dom-for="bandwidth"
            />
          </div>
        </div>

        <div class="row mb-1">
          <label class="col-form-label col-sm-6 pt-0" for="nchan"
            ># Channels</label
          >
          <div class="col-sm-6">
            <select
              class="form-select"
              id="nchan-select"
              name="nchan"
              aria-label="Select number of channels"
              disabled={!Boolean($data.bandwidth)}
            >
              {#each channels as channel}
                <option value={channel.value}>{channel.label}</option>
              {/each}
            </select>
            <div
              class="error-msg"
              id="nchan-validation"
              data-felte-reporter-dom-for="nchan"
            />
          </div>
        </div>

        <div class="row mb-1">
          <label class="col-sm-6 col-form-label" for="tint"
            >Integration Time</label
          >
          <div class="col-sm-6">
            <input
              class="form-control"
              type="text"
              inputmode="numeric"
              name="tint"
            />
            <div
              class="error-msg"
              id="tint-validation"
              data-felte-reporter-dom-for="tint"
            />
          </div>
        </div>

        <fieldset class="row mb-1">
          <legend class="col-form-label col-sm-6 pt-0"
            >Velocity reference</legend
          >
          <div class="col-sm-6">
            <div class="form-check">
              <input
                class="form-check-input"
                type="radio"
                name="vframe"
                id="vframe_lsrk"
                value="lsrk"
              />
              <label class="form-check-label" for="vframe_lsrk"> Lsrk </label>
            </div>
            <div class="form-check">
              <input
                class="form-check-input"
                type="radio"
                name="vframe"
                id="vframe_helio"
                value="helio"
              />
              <label class="form-check-label" for="vframe_helio"> Helio </label>
            </div>
            <div
              class="error-msg"
              id="vframe-validation"
              data-felte-reporter-dom-for="vframe"
            />
          </div>
        </fieldset>

        <fieldset class="row mb-1">
          <legend class="col-form-label col-sm-6 pt-0"
            >Specify velocity frame for Doppler shift</legend
          >
          <div class="col-sm-6">
            <div class="form-check">
              <input
                class="form-check-input"
                type="radio"
                name="framevdef"
                id="framevdef_lsrk"
                value="optical"
              />
              <label class="form-check-label" for="framevdef_optical">
                Optical
              </label>
            </div>
            <div class="form-check">
              <input
                class="form-check-input"
                type="radio"
                name="framevdef"
                id="framevdef_radio"
                value="radio"
              />
              <label class="form-check-label" for="framevdef_radio">
                Radio
              </label>
            </div>
            <div
              class="error-msg"
              id="framevdef-validation"
              data-felte-reporter-dom-for="framevdef"
            />
          </div>
        </fieldset>

        <div class="form-check form-switch">
          <input class="form-check-input" type="checkbox" name="use_cal" />
          <label class="form-check-label" for="use_cal"
            >Use the noise diode?</label
          >
        </div>

        {#if $data.use_cal}
          <fieldset class="row mb-1">
            <legend class="col-form-label col-sm-6 pt-0"
              >Strength of the noise diode</legend
            >
            <div class="col-sm-6">
              <div class="form-check">
                <input
                  class="form-check-input"
                  type="radio"
                  name="noisecal"
                  id="noisecal_high"
                  value="high"
                />
                <label class="form-check-label" for="noisecal_high">
                  High
                </label>
              </div>
              <div class="form-check">
                <input
                  class="form-check-input"
                  type="radio"
                  name="noisecal"
                  id="noisecal_low"
                  value="low"
                />
                <label class="form-check-label" for="noisecal_low"> Low </label>
              </div>
              <div
                class="error-msg"
                id="noisecal-validation"
                data-felte-reporter-dom-for="noisecal"
              />
            </div>
          </fieldset>
        {/if}
        <fieldset class="row mb-1">
          <legend class="col-form-label col-sm-6 pt-0"
            >Specify polarization state</legend
          >
          <div class="col-sm-6">
            <div class="form-check">
              <input
                class="form-check-input"
                type="radio"
                name="pol"
                id="pol_linear"
                value="linear"
              />
              <label class="form-check-label" for="pol_linear"> Linear </label>
            </div>
            <div class="form-check">
              <input
                class="form-check-input"
                type="radio"
                name="pol"
                id="pol_circular"
                value="circular"
              />
              <label class="form-check-label" for="pol_circular">
                Circular
              </label>
            </div>
            <div
              class="error-msg"
              id="pol-validation"
              data-felte-reporter-dom-for="pol"
            />
          </div>
        </fieldset>

        <div class="form-check form-switch">
          <input class="form-check-input" type="checkbox" name="xcor" />
          <label class="form-check-label" for="xcor"
            >Record Cross polarization as well?</label
          >
        </div>

        <button type="submit" class="btn btn-primary">Submit</button>
        <button type="reset" class="btn btn-warning" on:click={reset}
          >Clear</button
        >
        <button type="button" class="btn btn-info" on:click={genDummyData}
          >Insert Dummy Data</button
        >
      </form>
    </div>
    <div class="col-lg-3">
      <div class="row">
        <h6>Form Data</h6>
        <code><pre>{JSON.stringify($data, null, 2)}</pre></code>
      </div>

      <div class="row">
        <h6>Form Errors</h6>
        <code><pre>{JSON.stringify($errors, null, 2)}</pre></code>
      </div>
    </div>
    <div class="col-lg-3">
      <div class="row">
        <h6>Config String</h6>
        {#if $isValid}
          <code><pre>{genConfigString($data)}</pre></code>
        {:else}
          <p on:click={genDummyData} style="cursor: pointer">
            Complete the form to generate a config string (or click here to use
            fake data)
          </p>
        {/if}
      </div>
    </div>
  </div>
</main>

<style>
  .error-msg {
    width: 100%;
    margin-top: 0.25rem;
    font-size: 0.875em;
    color: #dc3545;
  }
</style>
