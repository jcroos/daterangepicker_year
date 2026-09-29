# daterangepicker_year

A jQuery date range picker with **year and month drill-down** in the custom range calendars.

`daterangepicker_year.js` and `daterangepicker_year.css` are built on top of
[daterangepicker](http://www.daterangepicker.com/) v3.0.3 by Dan Grossman (MIT licence).
All original options, ranges, locale settings and callbacks work the same way. The
difference is that you can jump to any year and month without clicking "previous month"
over and over.

## What it adds

When the picker opens it shows the normal day calendars. In the custom range calendars:

1. **Click the month title** (e.g. `Sep 2026`) to open the **year view**, a grid of 12 years
   (e.g. `2016 - 2027`). The ‹ › arrows move 12 years back or forward.
2. **Click a year** to open the **month view** for that year. The ‹ › arrows move one year
   back or forward. Click the year title to go back to the year view.
3. **Click a month** to go back to the day view for that month, then pick dates as usual.

Each calendar (left and right) drills down on its own. With `linkedCalendars: true` (the
default), picking a month on one side moves the other side to the next or previous month
and puts it back in the day view.

In the year and month views:

| Style        | Meaning                                                           |
|--------------|-------------------------------------------------------------------|
| Blue (active)| The year or month the calendar is showing now                     |
| Light blue   | Years or months inside the selected start–end range               |
| **Bold**     | The current year or month                                         |
| Greyed out   | Outside `minDate`/`maxDate` or `minYear`/`maxYear`; not clickable |

The picker always opens in the day view.

> **Note:** Drill-down is turned on only when `showDropdowns` is `false` (the default).
> With `showDropdowns: true`, the month title shows the month and year dropdowns instead,
> just like the original library.

## Project files

| File                        | Description                                               |
|-----------------------------|-----------------------------------------------------------|
| `daterangepicker_year.js`   | The new plugin with year and month drill-down             |
| `daterangepicker_year.css`  | Original styles plus styles for the year and month views  |
| `sample_year.html`          | Demo page for `daterangepicker_year`                      |
| `daterangepicker.js`        | Original daterangepicker v3.0.3, for reference            |
| `daterangepicker.css`       | Original daterangepicker styles, for reference            |
| `sample.html`               | Demo page for the original `daterangepicker`              |
| `jquery.min.js`             | jQuery (dependency)                                       |
| `moment.min.js`             | Moment.js (dependency)                                    |
| `bootstrap.min.css`         | Bootstrap (optional, used by the sample pages)            |

## Dependencies

- [jQuery](https://jquery.com/)
- [Moment.js](https://momentjs.com/)
- Bootstrap CSS (optional, for page layout only)

## Usage

Include the files:

```html
<script src="jquery.min.js"></script>
<script src="moment.min.js"></script>
<link href="daterangepicker_year.css" rel="stylesheet" />
<script src="daterangepicker_year.js"></script>
```

Set up the picker on an input:

```html
<input class="form-control" type="text" id="txtQryFromToDt" name="txtQryFromToDt">

<script>
  $(function () {
    $("#txtQryFromToDt").daterangepicker_year({
      ranges: {
        'Today'       : [moment(), moment()],
        'Yesterday'   : [moment().subtract(1, 'days'), moment().subtract(1, 'days')],
        'Last 7 Days' : [moment().subtract(6, 'days'), moment()],
        'Last 30 Days': [moment().subtract(29, 'days'), moment()],
        'This Month'  : [moment().startOf('month'), moment().endOf('month')],
        'Last Month'  : [moment().subtract(1, 'month').startOf('month'),
                         moment().subtract(1, 'month').endOf('month')]
      },
      locale: {
        format     : 'DD-MMM-YYYY',
        cancelLabel: 'Clear'
      },
      startDate : moment().startOf('month'),
      endDate   : moment().endOf('month'),
      autoApply : true
    });

    $("#txtQryFromToDt").on('apply.daterangepicker_year', function (ev, picker) {
      $(this).val(picker.startDate.format('DD-MMM-YYYY') + ' - ' +
                  picker.endDate.format('DD-MMM-YYYY'));
    });
  });
</script>
```

For a complete example, including "This Financial Year" and "Last Financial Year"
(April–March) ranges, see `sample_year.html`.

### Limiting the years

The year and month views use the standard options to decide which years and months can
be picked:

```js
$("#txtQryFromToDt").daterangepicker_year({
  minDate: moment('2015-04-01'),
  maxDate: moment().endOf('year'),
  minYear: 2000,   // default: 100 years before today
  maxYear: 2030    // default: 100 years after today
});
```

The ‹ › arrows are hidden when there is nothing more to show in that direction.

## Differences from `daterangepicker`

The plugin uses its own name, so both libraries can be loaded on the same page without
clashing.

| Item                    | `daterangepicker`                  | `daterangepicker_year`                  |
|-------------------------|------------------------------------|-----------------------------------------|
| jQuery plugin           | `$(el).daterangepicker(...)`       | `$(el).daterangepicker_year(...)`       |
| Instance data key       | `$(el).data('daterangepicker')`    | `$(el).data('daterangepicker_year')`    |
| Browser global          | `window.daterangepicker`           | `window.daterangepicker_year`           |
| Events                  | `apply.daterangepicker`, ...       | `apply.daterangepicker_year`, ...       |
| Event namespace         | `.daterangepicker`                 | `.daterangepicker_year`                 |
| Defaults                | `$.fn.daterangepicker.defaultOptions` | `$.fn.daterangepicker_year.defaultOptions` |

Events you can listen to: `show`, `hide`, `showCalendar`, `hideCalendar`, `apply`,
`cancel` and `outsideClick`, each followed by `.daterangepicker_year`.

The popup element still has the CSS class `daterangepicker`, so your existing theme or
style overrides still work. `daterangepicker_year.css` already includes all the original
styles, so you only need this one stylesheet.

### Code changes, for maintainers

In `daterangepicker_year.js`:

- `leftCalendar` / `rightCalendar` now store `viewMode` (`'days'`, `'months'` or `'years'`)
  and `yearPageStart` (the first year on the 12-year page).
- `renderCalendar()` calls the new `renderYearView()` or `renderMonthView()` when the
  calendar is not in the day view. Helper methods: `yearBounds()`, `pickerHeaderHtml()`
  and `pickerGridHtml()`.
- New click handlers: `clickSwitch()` (title opens the year view), `clickYear()` and
  `clickMonth()`.
- `clickPrev()` / `clickNext()` call `stepPicker()` first, so the arrows page through
  years in the year and month views.
- The month/year logic from `monthOrYearChanged()` was moved into a reusable
  `setCalendarMonth(side, month, year)`, which the dropdowns and `clickMonth()` both use.
- `show()` resets both calendars to the day view.
- `hoverDate()` ignores year and month cells when the other calendar is showing them.

In `daterangepicker_year.css`, new rules for `th.month.drp-switch`, `table.drp-grid`,
`td.drp-year` and `td.drp-month` (hover, `today`, `drp-in-sel` and `active` states).

## Running the demo

Open `sample_year.html` in a browser. No build step or server is needed. Click the
**Period** field, then click a month title in either calendar to try the drill-down.

## Changelog

### Initial release (2026-09-28)

Based on `daterangepicker` v3.0.3.

**Added**

- Year view: click the month title in a custom range calendar to see a 12-year grid.
  The ‹ › arrows move 12 years back or forward.
- Month view: click a year to see its 12 months, then click a month to go back to the
  day calendar. The ‹ › arrows move one year back or forward, and clicking the year title
  goes back to the year view.
- Highlights in the year and month views for the shown year or month, the selected range
  and the current year or month. Years and months outside `minDate`/`maxDate` or
  `minYear`/`maxYear` are greyed out.
- Styles for the new views in `daterangepicker_year.css`.
- `sample_year.html` demo page, with "This Financial Year" and "Last Financial Year"
  (April–March) ranges.

**Changed**

- Renamed the plugin to `daterangepicker_year`, along with its data key, browser global,
  events and event namespace, so it can run alongside the original `daterangepicker`.
- The picker always opens in the day view.
- With `linkedCalendars: true`, picking a month on one calendar puts the other calendar
  back in the day view.

## Credits and licence

Based on [daterangepicker](https://github.com/dangrossman/daterangepicker) v3.0.3,
Copyright (c) 2012-2018 Dan Grossman, released under the
[MIT licence](http://www.opensource.org/licenses/mit-license.php). The year and month
drill-down is an extension of that work and uses the same licence.
