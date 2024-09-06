---
title: How to Calculate the Exponential Decay Formula in C++
layout: note
tags:
  - Cpp
  - Qt6
---

To calculate the value for an amplifier based on the linear position of the slider, where the value doubles every 1/10th of the way, you can use an exponential function. Specifically, the relationship between the slider position \( p \) (ranging from 0.0 to 1.0) and the amplifier value \( v \) can be described using the following formula:

\[ v = 2^{10(p - 1)} \]

This formula ensures that at \( p = 1.0 \), \( v = 1 \); at \( p = 0.9 \), \( v = 0.5 \); at \( p = 0.8 \), \( v = 0.25 \), and so on.

Here's a breakdown of how this works:
- When \( p = 1.0 \):
  \[ v = 2^{10(1 - 1)} = 2^0 = 1 \]
- When \( p = 0.9 \):
  \[ v = 2^{10(0.9 - 1)} = 2^{-1} = 0.5 \]
- When \( p = 0.8 \):
  \[ v = 2^{10(0.8 - 1)} = 2^{-2} = 0.25 \]

And this pattern continues, where each 0.1 decrease in \( p \) results in halving the value of \( v \).

To implement this in your C++/Qt program, you can use the `pow` function from the `<cmath>` library to calculate the amplifier value based on the slider position. Here's an example code snippet:

```cpp
#include <cmath>
#include <QSlider>
#include <QObject>

class AmplifierControl : public QObject {
    Q_OBJECT

public:
    AmplifierControl(QSlider* slider) {
        connect(slider, &QSlider::valueChanged, this, &AmplifierControl::onSliderValueChanged);
    }

private slots:
    void onSliderValueChanged(int sliderValue) {
        // Assuming slider value ranges from 0 to 100
        double position = sliderValue / 100.0;
        double amplifierValue = pow(2, 10 * (position - 1));
        // Send amplifierValue to the amplifier
    }
};

// Usage
// QSlider* gainSlider = new QSlider(Qt::Horizontal);
// AmplifierControl* amplifierControl = new AmplifierControl(gainSlider);
```

In this code, the `onSliderValueChanged` slot converts the slider value (which ranges from 0 to 100) to a position between 0.0 and 1.0, calculates the corresponding amplifier value using the exponential formula, and then sends this value to the amplifier.

## Exponential Decay

The formula is a specific case of an exponential decay function. In general, exponential functions of the form \( v = a \cdot b^x \) describe processes where the rate of change of a quantity is proportional to the quantity itself. 

In your case, the function \( v = 2^{10(p - 1)} \) is a type of exponential decay function because the value \( v \) decreases exponentially as the position \( p \) decreases linearly. Specifically, your function models a geometric progression where each step results in halving the value, which is a common characteristic of exponential decay.

Here are the key characteristics:
- **Exponential Decay:** The value decreases by a constant factor (in this case, halving) over equal intervals of the independent variable (in this case, the slider position).
- **Geometric Progression:** Each step corresponds to a fixed ratio between successive values, which in your function is a factor of \( \frac{1}{2} \) for every 0.1 decrease in the slider position.

# Reading the Gain Value and Converting that to a Slider Position

To convert the gain level value back into the appropriate slider position, you need to find the inverse of the exponential decay function we used earlier. Given the function:

\[ v = 2^{10(p - 1)} \]

To find \( p \) in terms of \( v \), we can take the logarithm base 2 of both sides and solve for \( p \):

1. Start with the original equation:
\[ v = 2^{10(p - 1)} \]

2. Take the logarithm base 2 of both sides:
\[ \log_2(v) = \log_2(2^{10(p - 1)}) \]

3. Simplify using the property of logarithms that \( \log_b(b^x) = x \):
\[ \log_2(v) = 10(p - 1) \]

4. Solve for \( p \):
\[ \log_2(v) = 10p - 10 \]
\[ \log_2(v) + 10 = 10p \]
\[ p = \frac{\log_2(v) + 10}{10} \]

Since most programming languages, including C++, do not have a direct function for \( \log_2 \), you can use the change of base formula for logarithms, which is:

\[ \log_2(v) = \frac{\log(v)}{\log(2)} \]

Here's how you can implement this in C++/Qt:

```cpp
#include <cmath>
#include <QSlider>
#include <QObject>

class AmplifierControl : public QObject {
    Q_OBJECT

public:
    AmplifierControl(QSlider* slider) {
        connect(slider, &QSlider::valueChanged, this, &AmplifierControl::onSliderValueChanged);
    }

    void setSliderPositionFromGain(double gainValue) {
        double position = (std::log(gainValue) / std::log(2) + 10) / 10.0;
        int sliderValue = static_cast<int>(position * 100); // Assuming slider value ranges from 0 to 100
        slider->setValue(sliderValue);
    }

private slots:
    void onSliderValueChanged(int sliderValue) {
        // Assuming slider value ranges from 0 to 100
        double position = sliderValue / 100.0;
        double amplifierValue = std::pow(2, 10 * (position - 1));
        // Send amplifierValue to the amplifier
    }

private:
    QSlider* slider;
};

// Usage
// QSlider* gainSlider = new QSlider(Qt::Horizontal);
// AmplifierControl* amplifierControl = new AmplifierControl(gainSlider);
// amplifierControl->setSliderPositionFromGain(gainValue);
```

In this example, the `setSliderPositionFromGain` method converts the gain value back to the slider position and sets the slider's value accordingly. The `log` function from the `<cmath>` library is used along with the change of base formula to compute the logarithm base 2 of the gain value.