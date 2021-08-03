<style>
  :global(.error-msg) {
    width: 100%;
    margin-top: 0.25rem;
    font-size: 0.875em;
    color: #dc3545;
  }
</style>

<script lang="ts">
  import { createForm } from "felte";
  import reporter from "@felte/reporter-dom";
  import { validator } from "@felte/validator-yup";
  import * as yup from "yup";

  import ConfigTextField from "./ConfigTextField.svelte";
  import ConfigNumberField from "./ConfigNumberField.svelte";
  import ConfigBooleanField from "./ConfigBooleanField.svelte";
  import ConfigRadioField from "./ConfigRadioField.svelte";
  import ConfigSelectField from "./ConfigSelectField.svelte";

  import { Popover } from "sveltestrap";

  function validateRestFreq(value, context) {
    console.log("validating", value);
    try {
      value.split(",").forEach((freq) => {
        if (!freq || isNaN(Number(freq))) {
          console.log("freq", freq, "is not an number");
          throw Error("bad!");
        }
      });
    } catch (error) {
      return false;
    }

    return true;
  }
  const schema = yup.object({
    conf_name: yup.string().required(),
    receiver: yup.string().ensure().required(),
    nwin: yup.number().typeError("Must be a positive number").positive(),
    rest_freq: yup.string().ensure().test({
      name: "rest_freq",
      message: "Must be comma-separated numbers, e.g. '1.0,2,3.0'",
      test: validateRestFreq,
    }),
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
  const vegasModes = [
    // mode, bandwidth, # channels
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

  const noisecalItems = [
    { name: "High", value: "high" },
    { name: "Low", value: "low" },
  ];

  const polItems = [
    { name: "Linear", value: "Linear" },
    { name: "Circular", value: "Circular" },
  ];

  const velocityRefs = [
    { name: "Helio", value: "helio" },
    { name: "Bary", value: "bary" },
  ];

  const framevdefItems = [
    { name: "Optical", value: "optical" },
    { name: "Radio", value: "radio" },
  ];

  let channels = [];
  // Whenever the bandwidth field changes, update the available channels
  $: {
    channels = vegasModes
      .filter(([mode, bw, nchan]) => Number(bw) === Number($data.bandwidth))
      .map(([mode, bw, nchan]) => ({
        value: nchan,
        label: nchan,
      }));
  }

  function cleanBoolean(value: boolean): string {
    return value ? "True" : "False";
  }

  function cleanString(value: string): string {
    return `'${value}'`;
  }

  function cleanNumber(value: number): string {
    return JSON.stringify(value);
  }

  function cleanButActuallyDoNothing(value: string): string {
    return value;
  }

  const initialFormValues = {
    conf_name: "",
    receiver: "",
    obstype: "Spectroscopy",
    backend: "VEGAS",
    nwin: null,
    rest_freq: null,
    deltafreq: 0.0,
    dopplertrackfreq: 0.0,
    bandwidth: null,
    nchan: null,
    swmode: "tp",
    swtype: "none",
    swper: 1.0,
    swfreq: "0.0,0.0",
    tint: null,
    vframe: "",
    framevdef: "",
    use_cal: false,
    noisecal: "",
    pol: "",
    "vegas.vpol": "cross",
    notchfilter: "In",
    xcor: false,
  };

  const cleaners = {
    conf_name: cleanString,
    receiver: cleanString,
    nwin: cleanNumber,
    rest_freq: cleanButActuallyDoNothing,
    bandwidth: cleanNumber,
    nchan: cleanNumber,
    tint: cleanNumber,
    vframe: cleanString,
    framevdef: cleanString,
    use_cal: cleanBoolean,
    noisecal: cleanString,
    pol: cleanString,
    xcor: cleanBoolean,
    "vegas.vpol": cleanString,
    obstype: cleanString,
    backend: cleanString,
    deltafreq: cleanNumber,
    swmode: cleanString,
    swtype: cleanString,
    swper: cleanNumber,
    swfreq: cleanButActuallyDoNothing,
    notchfilter: cleanString,
    dopplertrackfreq: cleanNumber,
  };
  const { form, isValid, data, errors, setField, setFields, reset } =
    createForm({
      onSubmit: (data) => console.log("Submit", data),
      extend: [validator, reporter({ single: true })],
      validateSchema: schema,
      initialValues: initialFormValues,
    });

  function handleCopyToClipboard(text: string, message?: string): void {
    const promise = navigator.clipboard.writeText(text);
    promise.then(
      () => console.log("Copied to clipboard"),
      () => console.error("Failed to copy to clipboard")
    );
  }

  function genDummyData() {
    const _dummyData = {
      ...initialFormValues,
      conf_name: "dummy inputs",
      receiver: "Rcvr_340",
      nwin: 1,
      rest_freq: 1,
      bandwidth: 1080,
      nchan: 16385,
      tint: 1,
      vframe: "bary",
      framevdef: "radio",
      use_cal: false,
      noisecal: "high",
      pol: "Linear",
      xcor: true,
    };
    setFields(_dummyData);
    // TODO: Is there a away to avoid doing this manually?
    document.getElementById("receiver-select").value = _dummyData.receiver;
    document.getElementById("bandwidth-select").value = _dummyData.bandwidth;
    // TODO: This doesn't work; perhaps some race condition? Works if you click
    //       generate button twice
    console.log("channels", channels);
    console.log("Setting nchan value to", JSON.stringify(_dummyData.nchan));
    document.getElementById("nchan-select").value = "16385";
  }

  function genConfigString(data) {
    const normalizedConfName = data.conf_name.replace(" ", "_");
    const lines = [];
    Object.entries(data).forEach(([key, value]) => {
      // TODO: break this logic out into a function, or something else sensible.
      //       I guess we'll need some sort of dependency tree at some point
      // Do not add conf_name to config
      // Do not add notchfilter unless receiver is L-Band
      if (
        key !== "conf_name" &&
        !(key === "notchfilter" && data["receiver"] !== "Rcvr1_2")
      ) {
        lines.push(`  "${key}": ${cleaners[key](value)}`);
      }
    });
    return `${normalizedConfName}="""\n${lines.join("\n")}\n"""`;
  }

  let formData = [];
  $: {
    formData = Object.entries($data).map(([key, value]) => ({
      [key]: value,
      type: typeof value,
    }));
  }

  let finalConfigString = "";
  $: {
    finalConfigString = genConfigString($data);
  }
</script>

<main class="container">
  <h1>Generate AstrID Config</h1>
  <h2>Position Switched ON/OFF Observation</h2>
  <div class="row">
    <div class="col-lg-6">
      <form use:form>
        <ConfigTextField label="Name of the Config section" name="conf_name" />
        <ConfigSelectField
          label="Receiver"
          name="receiver"
          items="{receivers}"
        />
        <ConfigNumberField label="# of spectral windows" name="nwin" />
        <ConfigNumberField label="Rest Frequency" name="rest_freq" />
        <ConfigSelectField
          label="Bandwidth (MHz)"
          name="bandwidth"
          items="{bandwidths}"
        />
        <ConfigSelectField
          label="# Channels"
          name="nchan"
          items="{channels}"
          disabled="{!Boolean($data.bandwidth)}"
        />
        <ConfigNumberField label="Integration Time" name="tint" />
        <ConfigRadioField
          label="Velocity reference"
          name="vframe"
          items="{velocityRefs}"
        />
        <ConfigRadioField
          label="Specify velocity frame for Doppler shift"
          name="framevdef"
          items="{framevdefItems}"
        />
        <ConfigBooleanField label="Use the noise diode" name="use_cal" />
        {#if $data.use_cal}
          <ConfigRadioField
            label="Strength of the noise diode"
            name="noisecal"
            items="{noisecalItems}"
          />
        {/if}
        <ConfigRadioField
          label="Specify polarization state"
          name="pol"
          items="{polItems}"
        />
        <ConfigBooleanField label="Record Cross polarization" name="xcor" />
        <ConfigTextField label="Observation Type" name="obstype" hidden />
        <ConfigTextField label="Backend." name="backend" hidden />
        <ConfigNumberField label="Delta Freq." name="deltafreq" hidden />
        <ConfigTextField label="Switching Mode" name="swmode" hidden />
        <ConfigTextField label="Switching Type" name="swtype" hidden />
        <ConfigNumberField label="Switching Period" name="swper" hidden />
        <ConfigTextField label="Switching Freq." name="swfreq" hidden />
        <ConfigTextField label="Notch Filter" name="notchfilter" hidden />

        <button type="submit" class="btn btn-primary">Submit</button>
        <button type="reset" class="btn btn-warning" on:click="{reset}">
          Clear
        </button>
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
          <code><pre>{finalConfigString}</pre></code>
          <a
            class="btn btn-primary"
            type="button"
            tabindex="0"
            id="copy-script-to-clipboard-btn"
            on:click="{() => handleCopyToClipboard(finalConfigString)}"
          >
            Copy to Clipboard
          </a>
          <Popover
            target="copy-script-to-clipboard-btn"
            placement="top"
            dismissible
          >
            Done!
          </Popover>
        {:else}
          <p>Complete the form to generate a config string</p>
          <button
            class="btn btn-info"
            id="dummy-data-btn"
            on:click="{genDummyData}"
          >
            Generate Dummy Data
          </button>
        {/if}
      </div>
    </div>
  </div>
</main>
