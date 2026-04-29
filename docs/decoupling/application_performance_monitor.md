# ApplicationPerformanceMonitor Decoupling Report

## Modules Scanned

- `Magento_ApplicationPerformanceMonitor` — 11 PHP files
- `Magento_ApplicationPerformanceMonitorNewRelic` — 1 PHP file

## Can They Be Removed? YES — trivially

**These modules are already perfectly decoupled.** Zero external coupling exists.

---

## Architecture

### APM Base Module (11 files)

A self-contained profiling framework:
- `Profiler/Profiler.php` — orchestrator with input gatherers + output writers
- `Profiler/InputInterface.php` — contract for metric collectors
- `Profiler/OutputInterface.php` — contract for metric reporters
- `Profiler/Metric.php`, `MetricType.php`, `Metrics.php` — metric value objects
- `Profiler/MetricsGatherer.php` — collects metrics from inputs
- `Profiler/MetricsComparator.php` — compares before/after metrics
- `Profiler/Input/GeneralInput.php` — collects basic metrics (memory, time)
- `Profiler/Output/LoggerOutput.php` — writes metrics to log
- `Plugin/ApplicationPerformanceMonitor.php` — `aroundLaunch()` plugin on `Framework\AppInterface`

**Dependencies:** Only `magento/framework`. Zero module dependencies.

### APM NewRelic Module (1 file)

- `Profiler/Output/NewRelicOutput.php` — sends metrics to New Relic
- Adds itself as an output to the Profiler via di.xml

**Dependencies:** `magento/framework`, `magento/module-application-performance-monitor`, `magento/module-new-relic-reporting`

### How They Hook In

The base module registers a single plugin on `Magento\Framework\AppInterface::launch()`:

```xml
<type name="Magento\Framework\AppInterface">
    <plugin name="application-performance-monitor" 
            type="Magento\ApplicationPerformanceMonitor\Plugin\ApplicationPerformanceMonitor"/>
</type>
```

The `aroundLaunch()` plugin wraps the entire application execution with metric collection. If `$this->profiler->isEnabled()` returns false, it just calls `$proceed()` with zero overhead.

The NewRelic module simply adds an output writer to the profiler's DI arguments:

```xml
<type name="Magento\ApplicationPerformanceMonitor\Profiler\Profiler">
    <arguments>
        <argument name="outputs" xsi:type="array">
            <item name="NewRelicOutput" xsi:type="object">...NewRelicOutput</item>
        </argument>
    </arguments>
</type>
```

---

## Coupling Analysis

| Direction | Count |
|-----------|-------|
| Modules depending on APM (composer) | **0** |
| PHP imports from APM in other modules | **0** |
| XML refs to APM in other modules | **0** |
| APM depending on other modules | Framework only (base), + NewRelicReporting (NewRelic variant) |

---

## Verdict

**These are the cleanest modules in the entire Magento codebase.**

- Zero inbound coupling
- Minimal outbound coupling (just Framework)
- Clean input/output plugin architecture
- Can be disabled via `module:disable` with zero impact
- Can be removed from composer with zero impact

**No decoupling work needed.** These are already a model of how Magento modules should be built:
1. Self-contained functionality
2. Hooks into core via a single plugin
3. Extension point via DI (new inputs/outputs can be added by other modules)
4. Feature-flagged (disabled = zero overhead)

The only note: the `NewRelicReporting` dependency in the NewRelic variant is appropriate — it needs the New Relic extension and reporting configuration. If you wanted to fully remove NewRelicReporting, the APM NewRelic module would need to be removed too (or refactored to talk to the `ext-newrelic` extension directly).
