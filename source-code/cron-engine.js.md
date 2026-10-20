/\*\*  
 \* CronPulse Pro \- Universal Cron AST Parser, Evaluator & Natural Language Translator  
 \* Pure ES6+, Zero External Dependencies  
 \* Supports 5-field (UNIX standard), 6-field (Quartz/Spring/AWS), and 7-field (Enterprise Quartz with Year)  
 \*/

class CronEngine {  
  constructor() {  
    this.MONTH\_NAMES \= \['JAN', 'FEB', 'MAR', 'APR', 'MAY', 'JUN', 'JUL', 'AUG', 'SEP', 'OCT', 'NOV', 'DEC'\];  
    this.DAY\_NAMES \= \['SUN', 'MON', 'TUE', 'WED', 'THU', 'FRI', 'SAT'\];  
  }

  /\*\*  
   \* Parse and validate a cron string into an AST.  
   \*/  
  parse(cronString) {  
    if (\!cronString || typeof cronString \!== 'string') {  
      return { valid: false, error: 'Empty or non-string cron expression' };  
    }

    const parts \= cronString.trim().split(/\\s+/);  
    let secondsField \= '0';  
    let minuteField, hourField, domField, monthField, dowField, yearField \= '\*';  
    let format \= 'unix';

    if (parts.length \=== 5\) {  
      // Standard UNIX: minute, hour, day of month, month, day of week  
      \[minuteField, hourField, domField, monthField, dowField\] \= parts;  
      format \= 'unix\_5\_field';  
    } else if (parts.length \=== 6\) {  
      // Quartz/Spring/AWS 6-field: seconds, minute, hour, dom, month, dow  
      \[secondsField, minuteField, hourField, domField, monthField, dowField\] \= parts;  
      format \= 'quartz\_6\_field';  
    } else if (parts.length \=== 7\) {  
      // Enterprise Quartz 7-field: seconds, minute, hour, dom, month, dow, year  
      \[secondsField, minuteField, hourField, domField, monthField, dowField, yearField\] \= parts;  
      format \= 'enterprise\_7\_field';  
    } else {  
      return {  
        valid: false,  
        error: \`Expected 5, 6, or 7 fields but received \${parts.length} fields\`  
      };  
    }

    try {  
      const parsedSeconds \= this.\_parseField(secondsField, 0, 59);  
      const parsedMinutes \= this.\_parseField(minuteField, 0, 59);  
      const parsedHours \= this.\_parseField(hourField, 0, 23);  
      const parsedDom \= this.\_parseField(domField, 1, 31, true); // allows '?'  
      const parsedMonths \= this.\_parseField(monthField, 1, 12, false, this.MONTH\_NAMES);  
      const parsedDow \= this.\_parseField(dowField, 0, 7, true, this.DAY\_NAMES, true); // 0 or 7 \= Sunday  
      const parsedYears \= this.\_parseField(yearField, 2020, 2099);

      return {  
        valid: true,  
        format,  
        raw: cronString.trim(),  
        fields: {  
          seconds: { raw: secondsField, values: parsedSeconds },  
          minutes: { raw: minuteField, values: parsedMinutes },  
          hours: { raw: hourField, values: parsedHours },  
          dayOfMonth: { raw: domField, values: parsedDom },  
          month: { raw: monthField, values: parsedMonths },  
          dayOfWeek: { raw: dowField, values: parsedDow },  
          year: { raw: yearField, values: parsedYears }  
        }  
      };  
    } catch (err) {  
      return { valid: false, error: err.message };  
    }  
  }

  /\*\*  
   \* Internal helper to parse individual cron field (ranges, steps, lists).  
   \*/  
  \_parseField(expr, min, max, allowQuestion \= false, aliasArray \= null, isDow \= false) {  
    if (allowQuestion && (expr \=== '?' || expr \=== '\*')) {  
      return this.\_range(min, max);  
    }  
    if (expr \=== '\*') {  
      return this.\_range(min, max);  
    }

    let normalized \= expr.toUpperCase();  
    if (aliasArray) {  
      aliasArray.forEach((alias, idx) \=\> {  
        const val \= aliasArray \=== this.MONTH\_NAMES ? idx \+ 1 : idx;  
        normalized \= normalized.split(alias).join(String(val));  
      });  
    }

    const values \= new Set();  
    const subParts \= normalized.split(',');

    for (const part of subParts) {  
      if (part.includes('/')) {  
        // Step syntax: "\*/15" or "10-50/5"  
        const \[rangePart, stepStr\] \= part.split('/');  
        const step \= parseInt(stepStr, 10);  
        if (isNaN(step) || step \<= 0\) throw new Error(\`Invalid step value in '\${part}'\`);

        let start \= min;  
        let end \= max;  
        if (rangePart \!== '\*' && rangePart \!== '') {  
          if (rangePart.includes('-')) {  
            const \[sStr, eStr\] \= rangePart.split('-');  
            start \= parseInt(sStr, 10);  
            end \= parseInt(eStr, 10);  
          } else {  
            start \= parseInt(rangePart, 10);  
          }  
        }  
        for (let i \= start; i \<= end; i \+= step) {  
          if (i \>= min && i \<= max) values.add(i);  
        }  
      } else if (part.includes('-')) {  
        // Range syntax: "1-5"  
        const \[sStr, eStr\] \= part.split('-');  
        const start \= parseInt(sStr, 10);  
        const end \= parseInt(eStr, 10);  
        if (isNaN(start) || isNaN(end) || start \> end) {  
          throw new Error(\`Invalid range specification in '\${part}'\`);  
        }  
        for (let i \= start; i \<= end; i++) {  
          if (i \>= min && i \<= max) values.add(i);  
        }  
      } else {  
        // Exact integer: "5"  
        let val \= parseInt(part, 10);  
        if (isNaN(val)) throw new Error(\`Non-numeric character in field: '\${part}'\`);  
        if (isDow && val \=== 7\) val \= 0; // Normalize 7 to 0 (Sunday)  
        if (val \< min || val \> max) {  
          throw new Error(\`Value \${val} out of bounds (\${min}-\${max})\`);  
        }  
        values.add(val);  
      }  
    }

    return Array.from(values).sort((a, b) \=\> a \- b);  
  }

  \_range(start, end) {  
    const arr \= \[\];  
    for (let i \= start; i \<= end; i++) arr.push(i);  
    return arr;  
  }

  /\*\*  
   \* Generates a fluent human-readable English explanation.  
   \*/  
  explain(ast) {  
    if (\!ast || \!ast.valid) return 'Invalid cron expression';

    const { seconds, minutes, hours, dayOfMonth, month, dayOfWeek } \= ast.fields;

    // 1\. Frequency summary  
    let timeDesc \= '';  
    if (seconds.raw \!== '0' && seconds.raw \!== '\*') {  
      timeDesc \= \`At second \${seconds.raw}, \`;  
    }

    if (minutes.raw.startsWith('\*/')) {  
      const step \= minutes.raw.split('/')\[1\];  
      timeDesc \+= \`Every \${step} minute\${step \> 1 ? 's' : ''}\`;  
    } else if (minutes.raw \=== '\*') {  
      timeDesc \+= 'Every minute';  
    } else {  
      const mins \= minutes.values.map(m \=\> String(m).padStart(2, '0')).join(', ');  
      timeDesc \+= \`At minute \${mins}\`;  
    }

    // 2\. Hour description  
    if (hours.raw.startsWith('\*/')) {  
      const step \= hours.raw.split('/')\[1\];  
      timeDesc \+= \`, every \${step} hour\${step \> 1 ? 's' : ''}\`;  
    } else if (hours.raw \!== '\*') {  
      const hrs \= hours.values.map(h \=\> {  
        const ampm \= h \>= 12 ? 'PM' : 'AM';  
        const displayHr \= h % 12 \=== 0 ? 12 : h % 12;  
        return \`\${displayHr}:00 \${ampm}\`;  
      }).join(', ');  
      timeDesc \+= \`, past hour(s) \${hrs}\`;  
    }

    // 3\. Day of month & Month  
    let dateDesc \= '';  
    if (dayOfMonth.raw \!== '\*' && dayOfMonth.raw \!== '?') {  
      dateDesc \+= \`, on day \${dayOfMonth.raw} of the month\`;  
    }

    if (month.raw \!== '\*') {  
      const mNames \= month.values.map(m \=\> this.MONTH\_NAMES\[m \- 1\]).join(', ');  
      dateDesc \+= \`, in \${mNames}\`;  
    }

    // 4\. Day of week  
    let dowDesc \= '';  
    if (dayOfWeek.raw \!== '\*' && dayOfWeek.raw \!== '?') {  
      const dNames \= dayOfWeek.values.map(d \=\> this.DAY\_NAMES\[d % 7\]).join(', ');  
      dowDesc \+= \`, only on \${dNames}\`;  
    }

    return \`\${timeDesc}\${dateDesc}\${dowDesc}.\`.trim();  
  }

  /\*\*  
   \* Calculates the next N execution dates starting from a baseline date.  
   \*/  
  getNextExecutions(ast, count \= 10, startDate \= new Date()) {  
    if (\!ast || \!ast.valid) return \[\];

    const results \= \[\];  
    const current \= new Date(startDate.getTime());  
    // Start search from next second  
    current.setSeconds(current.getSeconds() \+ 1);  
    current.setMilliseconds(0);

    const secVals \= new Set(ast.fields.seconds.values);  
    const minVals \= new Set(ast.fields.minutes.values);  
    const hrVals \= new Set(ast.fields.hours.values);  
    const domVals \= new Set(ast.fields.dayOfMonth.values);  
    const moVals \= new Set(ast.fields.month.values);  
    const dowVals \= new Set(ast.fields.dayOfWeek.values);  
    const yrVals \= new Set(ast.fields.year.values);

    let iterations \= 0;  
    const maxIterations \= 500000; // Protection ceiling

    while (results.length \< count && iterations \< maxIterations) {  
      iterations++;

      const year \= current.getFullYear();  
      if (\!yrVals.has(year)) {  
        current.setFullYear(year \+ 1, 0, 1);  
        current.setHours(0, 0, 0, 0);  
        continue;  
      }

      const month \= current.getMonth() \+ 1; // 1-12  
      if (\!moVals.has(month)) {  
        current.setMonth(current.getMonth() \+ 1, 1);  
        current.setHours(0, 0, 0, 0);  
        continue;  
      }

      const dom \= current.getDate();  
      const dow \= current.getDay(); // 0-6

      const domMatch \= domVals.has(dom);  
      const dowMatch \= dowVals.has(dow);

      let dayMatches \= false;  
      if (ast.fields.dayOfMonth.raw \=== '?' || ast.fields.dayOfMonth.raw \=== '\*') {  
        dayMatches \= dowMatch;  
      } else if (ast.fields.dayOfWeek.raw \=== '?' || ast.fields.dayOfWeek.raw \=== '\*') {  
        dayMatches \= domMatch;  
      } else {  
        dayMatches \= domMatch && dowMatch;  
      }

      if (\!dayMatches) {  
        current.setDate(current.getDate() \+ 1);  
        current.setHours(0, 0, 0, 0);  
        continue;  
      }

      const hour \= current.getHours();  
      if (\!hrVals.has(hour)) {  
        current.setHours(hour \+ 1, 0, 0, 0);  
        continue;  
      }

      const minute \= current.getMinutes();  
      if (\!minVals.has(minute)) {  
        current.setMinutes(minute \+ 1, 0, 0);  
        continue;  
      }

      const second \= current.getSeconds();  
      if (\!secVals.has(second)) {  
        current.setSeconds(second \+ 1);  
        continue;  
      }

      // Match found  
      results.push(new Date(current.getTime()));  
      current.setSeconds(current.getSeconds() \+ 1);  
    }

    return results;  
  }  
}

if (typeof module \!== 'undefined' && module.exports) {  
  module.exports \= CronEngine;  
}  
