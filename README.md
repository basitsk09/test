import React, { useEffect, useRef, useState, useMemo, useCallback } from 'react';
import { TextField } from '@mui/material';
import { TYPE_VALIDATION } from '../../constants/ValidationConstants';
import useCustomSnackbar from '../../hooks/useCustomSnackbar';
import { useTheme } from '@mui/material/styles';
import _ from 'lodash';

const FormInput = React.memo(
  ({
    name = '',
    label = '',
    value = '',
    onChange = () => {},
    onBlur = () => {},
    readOnly = false,
    error = false,
    helperText = '',
    inputType = 'amountDecimal', //Input type should be from TYPE_VALIDATION -> /constants/ValidationConstants
    focus = false,
    maxLength = 18,
    validateOnlyOnBlur = false,
    customStyles = {},
    debounceDuration = 850,
    ...props
  }) => {
    const inputRef = useRef(null);
    const snackbar = useCustomSnackbar();
    const theme = useTheme();
    const [tempValue, setTempValue] = useState(value);
    const leftAlignedInputTypes = useMemo(
      () =>
        new Set([
          'text',
          'inputValue',
          'alphaInput',
          'alphaNumeric',
          'splAlphaNumeric',
          'emailId',
          'alphaNumWithSpaceTrim',
          'alphaNumericWithSpace',
          'certainSpecialChars',
          'address',
          'time',
          'date-DD-MM-YYYY',
          'date-YYYY-MM-DD',
          'noSpecialChar',
          'panNumber',
          'tanNumber',
          'branchCode',
        ]),
      []
    );

    useEffect(() => {
      if (focus && inputRef.current) {
        inputRef.current.focus();
        inputRef.current.scrollIntoView({ behavior: 'smooth', block: 'center' });
        inputRef.current.style.outline = '2px solid red';
        inputRef.current.style.borderRadius = theme.shape.borderRadius.toString() + 'px';
        const timer = setTimeout(() => {
          if (inputRef.current) {
            inputRef.current.style.outline = '';
            inputRef.current.style.borderRadius = '';
          }
        }, 3000);
        return () => clearTimeout(timer);
      }
    }, [focus, theme.shape.borderRadius]);

    useEffect(() => {
      setTempValue(value);
    }, [value]);

    // Memoize the styles
    const inputStyles = useMemo(
      () => ({
        '& .MuiOutlinedInput-root': {
          '&.Mui-focused fieldset': {
            borderColor: error ? 'red' : 'inherit',
          },
        },
        ...(error && {
          '& .MuiOutlinedInput-notchedOutline': {
            borderColor: 'red',
          },
        }),
      }),
      [error]
    );

    // Memoize the debounced function
    const debouncedHandleChange = useMemo(() => {
      return _.debounce((event) => {
        onChange(event);
      }, debounceDuration);
    }, [onChange, debounceDuration]);

    const handleChange = (event) => {
      const { value } = event.target;

      if (!validateOnlyOnBlur) {
        const result = TYPE_VALIDATION(inputType, value, '');
        if (result === '') {
          setTempValue(value);
          debouncedHandleChange(event);
        } else {
          snackbar(result, 'error');
          return;
        }
      } else {
        setTempValue(value);
        debouncedHandleChange(event);
      }
    };

    const onBlurHandler = useCallback((event) => {
      const { value } = event.target;

      if (!validateOnlyOnBlur) {
        return onBlur(event);
      }

      const result = TYPE_VALIDATION(inputType, value, '');
      if (result === '') {
        return onBlur(event);
      }
      snackbar(result, 'error');
      return;
      // Clear the input value if validation fails
      //event.target.value = ''; // Clear the input field
      // handleChange(event);
    }, []);

    const getTextAlignment = useMemo(() => {
      return (inputType) => {
        return leftAlignedInputTypes.has(inputType) ? 'left' : 'right';
      };
    }, [leftAlignedInputTypes]);

    return (
      <TextField
        inputRef={inputRef}
        fullWidth
        type="text"
        inputMode={inputType === 'amountDecimal' ? 'decimal' : 'text'}
        name={name}
        label={label}
        value={readOnly ? value : tempValue ?? ''}
        onChange={handleChange}
        onBlur={onBlurHandler}
        slotProps={{
          htmlInput: {
            maxLength: inputType === 'wholeAmountDecimal' ? 19 : maxLength,
            readOnly: readOnly,
            sx: { textAlign: getTextAlignment(inputType) },
          },
        }}
        disabled={readOnly}
        error={error}
        helperText={error ? helperText : ''}
        size="small"
        sx={{ ...inputStyles, ...customStyles }}
        {...props}
      />
    );
  }
);

export default FormInput;
