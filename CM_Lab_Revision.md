# CM MATLAB Lab -- External Exam Revision

A compact revision guide containing the method, a one-line purpose,
MATLAB program, algorithm, and flowchart for each topic.

> **Note:** The programs below follow the faculty versions and the
> versions discussed during preparation. A few programs were
> reconstructed in the same style where a complete faculty code was not
> available (Newton's Law of Cooling, homogeneous ODE, and
> Newton--Raphson). Check the exact variable names and input format
> expected by your faculty.

------------------------------------------------------------------------

## 1. Jacobi Method

**What it does:** Solves a system of linear equations iteratively,
calculating every new variable using only values from the previous
iteration.

### MATLAB code

``` matlab
clc;
clear;
close all;

A = [8 -3 2; 4 -11 -1; 6 3 12];
B = [20; 33; 35];

X = zeros(size(B));
X_New = zeros(size(B));

tol = 1e-8;
max_iterations = 100;
n = length(B);

% Check diagonal dominance
for i = 1:n
    if abs(A(i,i)) <= sum(abs(A(i,:))) - abs(A(i,i))
        error('Matrix is not diagonally dominant');
    end
end

for k = 1:max_iterations
    X_Old = X;

    for i = 1:n
        sum_val = B(i) ...
            - A(i,1:i-1)*X_Old(1:i-1) ...
            - A(i,i+1:n)*X_Old(i+1:n);

        X_New(i) = sum_val/A(i,i);
    end

    X = X_New;
    err = norm(X-X_Old,inf);

    fprintf('%d\t',k);
    disp(X');

    if err < tol
        break;
    end
end

disp('Solution:');
disp(X);
```

### Algorithm

1.  Start and input coefficient matrix `A` and constant vector `B`.
2.  Initialize the solution vector with zeros.
3.  Check whether the matrix is diagonally dominant.
4.  Store the previous iteration values in `X_Old`.
5.  Calculate every new variable using only `X_Old` values.
6.  Calculate the error using the infinity norm.
7.  If the error is below tolerance, stop; otherwise repeat.
8.  Display the solution and stop.

### Flowchart

``` text
 START
   |
 Input A, B, tolerance and maximum iterations
   |
 Initialize X = 0
   |
 Check diagonal dominance
   |
 Store old values X_Old
   |
 Calculate all new X values using X_Old
   |
 Calculate error = norm(X - X_Old, infinity)
   |
 Error < tolerance?
   | Yes                  | No
 Display solution         Repeat iteration
   |
  STOP
```

------------------------------------------------------------------------

## 2. Gauss--Seidel Method

**What it does:** Solves a system of linear equations iteratively,
immediately using newly calculated values within the same iteration.

### MATLAB code

``` matlab
clc;
clear;
close all;

A = [8 -3 2; 4 -11 -1; 6 3 12];
B = [20; 33; 35];

X = zeros(size(B));
X_New = zeros(size(B));

tol = 1e-8;
max_iterations = 100;
n = length(B);

% Check diagonal dominance
for i = 1:n
    if abs(A(i,i)) <= sum(abs(A(i,:))) - abs(A(i,i))
        error('Matrix is not diagonally dominant');
    end
end

for k = 1:max_iterations
    X_Old = X;

    for i = 1:n
        sum_val = B(i) ...
            - A(i,1:i-1)*X(1:i-1) ...
            - A(i,i+1:n)*X_Old(i+1:n);

        X(i) = sum_val/A(i,i);
    end

    err = norm(X-X_Old,inf);
    fprintf('%d\t',k);
    disp(X');

    X_New = X;

    if err < tol
        break;
    end
end

disp('Solution:');
disp(X_New);
```

### Algorithm

1.  Start and input `A` and `B`.
2.  Initialize the solution vector with zeros.
3.  Check diagonal dominance.
4.  Store the previous iteration values in `X_Old`.
5.  Calculate each variable using the latest available values: new
    values for already updated variables and old values for the
    remaining variables.
6.  Calculate the error.
7.  If the error is below tolerance, stop; otherwise repeat.
8.  Display the solution and stop.

### Flowchart

``` text
 START
   |
 Input A, B, tolerance and maximum iterations
   |
 Initialize X = 0
   |
 Check diagonal dominance
   |
 Store old values X_Old
   |
 Calculate each X(i) using new and old values
   |
 Calculate error = norm(X - X_Old, infinity)
   |
 Error < tolerance?
   | Yes                  | No
 Display solution         Repeat iteration
   |
  STOP
```

------------------------------------------------------------------------

## 3. Bisection Method

**What it does:** Finds a root of an equation by repeatedly halving an
interval in which the function changes sign.

### MATLAB code

``` matlab
clc;
clear;
close all;

f = @(x) x^2 - 4;
a = 1;
b = 3;
tol = 0.0001;

if f(a)*f(b) >= 0
    disp('Invalid interval');
else
    while abs(b-a)/2 > tol
        c = (a+b)/2;

        if abs(f(c)) < tol
            break;
        end

        if f(a)*f(c) < 0
            b = c;
        else
            a = c;
        end
    end

    c = (a+b)/2;
    disp('Root is:');
    disp(c);
end
```

### Algorithm

1.  Start and define the function, interval endpoints, and tolerance.
2.  Check whether `f(a)` and `f(b)` have opposite signs.
3.  If not, display an invalid interval message.
4.  Calculate the midpoint `c = (a+b)/2`.
5.  If `abs(f(c))` is below tolerance, stop the loop.
6.  Otherwise, replace the endpoint that has the same sign as `f(c)`.
7.  Repeat until the interval half-width is within tolerance.
8.  Display the midpoint as the approximate root and stop.

### Flowchart

``` text
 START
   |
 Define f(x), a, b and tolerance
   |
 Is f(a)*f(b) < 0?
   | No                  | Yes
 Invalid interval        Calculate midpoint c
 Stop                       |
                         Is abs(f(c)) < tolerance?
                           | Yes          | No
                           |              Update interval
                           |              using sign test
                           |                 |
                           |              Repeat
                           |
                      Display root
                           |
                          STOP
```

------------------------------------------------------------------------

## 4. Exact and Non-Exact Differential Equations

**What it does:** Solves exact equations directly and handles certain
non-exact equations by finding an integrating factor depending only on
`x` or only on `y`.

### MATLAB code

``` matlab
clc;
clear;
syms x y

M = input('Enter M(x,y): ');
N = input('Enter N(x,y): ');

My = diff(M,y);
Nx = diff(N,x);

if simplify(My-Nx) == 0
    mu = 1;
else
    A = simplify((My-Nx)/N);
    B = simplify((Nx-My)/M);

    if ~has(A,y)
        mu = exp(int(A,x));
    elseif ~has(B,x)
        mu = exp(int(B,y));
    else
        disp('No standard integrating factor found');
        return;
    end
end

M1 = simplify(mu*M);
N1 = simplify(mu*N);

F = int(M1,x);
G = int(N1-diff(F,y),y);

sol = simplify(F+G);

disp('Integrating factor:')
disp(mu)
disp('Final solution F(x,y) = C:')
disp(sol)
```

### Algorithm

1.  Start and define symbolic variables `x` and `y`.
2.  Input `M(x,y)` and `N(x,y)`.
3.  Calculate `My` and `Nx`.
4.  If `My = Nx`, set integrating factor `mu = 1`.
5.  Otherwise, calculate the expressions used to test for an integrating
    factor depending only on `x` or only on `y`.
6.  Calculate the integrating factor if either standard condition is
    satisfied; otherwise display a message and stop.
7.  Multiply `M` and `N` by the integrating factor.
8.  Integrate to obtain the solution.
9.  Display the integrating factor and final solution.
10. Stop.

### Flowchart

``` text
 START
   |
 Input M(x,y), N(x,y)
   |
 Calculate My and Nx
   |
 Is My = Nx?
   | Yes                  | No
 Set mu = 1          Calculate A and B
                           |
                  Is A independent of y?
                    | Yes       | No
                 IF in x    Is B independent of x?
                               | Yes       | No
                            IF in y    Display no standard
                                       IF and stop
   |
 Form M1 = mu*M and N1 = mu*N
   |
 Integrate to obtain solution
   |
 Display integrating factor and solution
   |
  STOP
```

> This combined program covers exact equations and non-exact equations
> for which the standard integrating factor depends only on `x` or only
> on `y`. It does not solve every possible non-exact equation.

------------------------------------------------------------------------

## 5. Eigenvalues and Eigenvectors

**What it does:** Finds the eigenvalues and eigenvectors of a square
matrix using MATLAB's `eig()` function.

### MATLAB code

``` matlab
clc;
clear;
close all;

A = input("Enter the matrix: ");
[m,n] = size(A);

if(m~=n)
    error("Matrix must be square!")
end

[V,D] = eig(A);
disp("Eigen Vectors: ");
disp(V);
disp("Eigen Values: ");
disp(diag(D));
```

### Algorithm

1.  Start and input matrix `A`.
2.  Find the number of rows and columns.
3.  Check whether the matrix is square.
4.  If it is not square, display an error and stop.
5.  Find eigenvectors and eigenvalues using `eig(A)`.
6.  Display matrix `V` containing eigenvectors and diagonal entries of
    `D` containing eigenvalues.
7.  Stop.

### Flowchart

``` text
 START
   |
 Input matrix A
   |
 Find rows m and columns n
   |
 Is m = n?
   | No                  | Yes
 Display error           Calculate [V,D] = eig(A)
 Stop                       |
                         Display V and diag(D)
                              |
                             STOP
```

------------------------------------------------------------------------

## 6. Singular Value Decomposition (SVD)

**What it does:** Decomposes a matrix into three matrices using
`A = U*S*V'`.

### MATLAB code

``` matlab
clc;
clear;
close all;

% Read matrix
A = input('Enter the matrix: ');
[m,n] = size(A);

% Calculate A transpose * A
B = A' * A;

% Find eigenvalues and eigenvectors
[V,D] = eig(B);

% Sort eigenvalues in descending order
[eigenvalues,index] = sort(diag(D),'descend');
V = V(:,index);

% Calculate singular values
sigma = sqrt(max(eigenvalues,0));

% Create S matrix
S = zeros(m,n);
for i = 1:min(m,n)
    S(i,i) = sigma(i);
end

% Calculate U matrix
U = zeros(m,m);
r = min(m,n);

for i = 1:r
    if sigma(i) > 1e-6
        U(:,i) = A*V(:,i)/sigma(i);
    end
end

if rank(A) < min(m,n)
    [U, S, V] = svd(A);
end

% for i = r+1:m
%     [Q, ~] = qr(U);
%     U(:, i) = Q(:, i);
% end
%
% Display matrices
disp('U ='); disp(U);
disp('S ='); disp(S);
disp('V ='); disp(V);

% Reconstruct matrix
A1 = U*S*V';
disp('Reconstructed matrix =');
disp(A1);

disp('Error =');
disp(norm(A-A1));
```

### Algorithm

1.  Start and input matrix `A`.
2.  Calculate `B = A'*A`.
3.  Find eigenvalues and eigenvectors of `B`.
4.  Sort eigenvalues in descending order and rearrange eigenvectors.
5.  Calculate singular values by taking square roots of non-negative
    eigenvalues.
6.  Construct matrix `S` using the singular values.
7.  Calculate columns of `U` using `u_i = A*v_i/sigma_i`.
8.  If the matrix is rank deficient, use MATLAB's built-in `svd(A)` as
    in the program.
9.  Display `U`, `S`, and `V`.
10. Reconstruct `A1 = U*S*V'` and calculate reconstruction error.
11. Stop.

### Flowchart

``` text
 START
   |
 Input matrix A
   |
 Calculate B = A'*A
   |
 Find eigenvalues and eigenvectors
   |
 Sort eigenvalues and eigenvectors
   |
 Calculate singular values
   |
 Construct S and calculate U
   |
 Is matrix rank deficient?
   | Yes                 | No
 Use built-in SVD        Continue
   |
 Display U, S and V
   |
 Reconstruct A1 = U*S*V'
   |
 Calculate reconstruction error
   |
  STOP
```

------------------------------------------------------------------------

## 7. Principal Component Analysis (PCA)

**What it does:** Transforms data into principal components ordered by
the amount of variance they explain.

### MATLAB code

``` matlab
clc;
clear;
close all;

% Step 1: Read the original data matrix
A = input("Enter the array: ");
disp('Matrix A:');
disp(A);

% Step 2: Calculate the mean of each column
meanA = mean(A);

% Step 3: Center the data by subtracting column means
centeredA = A - meanA;
disp('Centered Data:');
disp(centeredA);

% Step 4: Calculate the covariance matrix of centered data
C = cov(centeredA);
disp('Covariance Matrix:');
disp(C);

% Step 5: Find eigenvalues and eigenvectors of covariance matrix
[V, D] = eig(C);

% Step 6: Sort eigenvalues in descending order
[eigenValues, index] = sort(diag(D), 'descend');

% Rearrange eigenvectors to match the sorted eigenvalues
V = V(:, index);

disp('Eigen Values:');
disp(eigenValues);
disp('Eigen Vectors:');
disp(V);

% Step 7: Project centered data onto principal components
Y = centeredA * V;
disp('Projected Data:');
disp(Y);

% Step 8: Calculate percentage of variance explained by each component
variance = eigenValues / sum(eigenValues) * 100;
disp('Variance Explained:');
disp(variance);

% Step 9: Plot the projected data
figure;
scatter(Y(:,1), Y(:,2), 60, 'filled');
xlabel('First Principal Component');
ylabel('Second Principal Component');
title('PCA Projection');
grid on;
```

### Algorithm

1.  Start and input the original data matrix.
2.  Calculate the mean of each column.
3.  Center the data by subtracting column means.
4.  Calculate the covariance matrix.
5.  Find eigenvalues and eigenvectors of the covariance matrix.
6.  Sort eigenvalues in descending order and rearrange eigenvectors.
7.  Project centered data onto principal components.
8.  Calculate the percentage of variance explained by each component.
9.  Plot the first two projected components.
10. Stop.

### Flowchart

``` text
 START
   |
 Input data matrix A
   |
 Calculate column means
   |
 Center the data
   |
 Calculate covariance matrix
   |
 Find eigenvalues and eigenvectors
   |
 Sort eigenvalues and eigenvectors
   |
 Project data onto principal components
   |
 Calculate variance explained
   |
 Plot projected data
   |
  STOP
```

------------------------------------------------------------------------

## 8. Exponential Growth and Decay

**What it does:** Calculates the growth or decay constant and estimates
the population at a later time using an exponential model.

### MATLAB code

``` matlab
clc;
clear;

N0 = 2000;
t1 = 3;
N1 = 5000;
k = log(N1 / N0) / t1;
t2 = 6;
N6 = N0 * exp(k * t2);

fprintf('Growth constant k = %.6f per hour\n', k);
fprintf('Estimated population after %d hours: %.0f bacteria\n', t2, N6);
```

### Algorithm

1.  Start.
2.  Initialize initial population `N0`, known time `t1`, and population
    `N1`.
3.  Calculate growth/decay constant `k`.
4.  Set the required time `t2`.
5.  Calculate the population using `N(t) = N0*exp(k*t)`.
6.  Display the constant and estimated population.
7.  Stop.

### Flowchart

``` text
 START
   |
 Initialize N0, t1 and N1
   |
 Calculate k = log(N1/N0)/t1
   |
 Set required time t2
   |
 Calculate N(t2) = N0*exp(k*t2)
   |
 Display k and estimated population
   |
  STOP
```

> If `N1 > N0`, `k` is positive and the model represents growth. If
> `N1 < N0`, `k` is negative and the model represents decay.

------------------------------------------------------------------------

## 9. Newton's Law of Cooling

**What it does:** Estimates the temperature of an object as it
approaches the surrounding temperature over time.

### MATLAB code

``` matlab
clc;
clear;

T0 = 100;  % Initial temperature
Ts = 25;   % Surrounding temperature
t1 = 5;    % Time
T1 = 60;   % Temperature after 5 minutes

k = -1/t1 * log((T1-Ts)/(T0-Ts));

t2 = 10;
T2 = Ts + (T0-Ts) * exp(-k*t2);

fprintf('Cooling constant k = %.6f per minute\n', k);
fprintf('Temperature after %d minutes: %.2f degrees\n', t2, T2);
```

### Algorithm

1.  Start.
2.  Initialize initial temperature `T0`, surrounding temperature `Ts`,
    known time `t1`, and temperature `T1`.
3.  Calculate cooling constant `k`.
4.  Set the required time `t2`.
5.  Calculate temperature using Newton's Law of Cooling.
6.  Display cooling constant and final temperature.
7.  Stop.

### Flowchart

``` text
 START
   |
 Initialize T0, Ts, t1 and T1
   |
 Calculate cooling constant k
   |
 Set required time t2
   |
 Calculate T2 = Ts + (T0-Ts)*exp(-k*t2)
   |
 Display k and T2
   |
  STOP
```

------------------------------------------------------------------------

## 10. Homogeneous Differential Equation

**What it does:** Finds the general and particular solutions of a
homogeneous differential equation using MATLAB's `dsolve()`.

### MATLAB code

``` matlab
clc;
clear;
format compact;
close all;

syms y(x)

ode = diff(y,x,2) - 5*(diff(y,x)) + 6*y == 0;

% General Solution
y_gen = dsolve(ode);
disp("General Solution:");
disp(y_gen);

% Initial Conditions
cond1 = y(0) == 1;
Dy = diff(y,x);

cond2 = Dy(0) == 0;

% Particular Solution
y_part = dsolve(ode, cond1, cond2);
disp("Particular Solution:");
disp(y_part);
```

### Algorithm

1.  Start and define symbolic function `y(x)`.
2.  Enter the homogeneous differential equation.
3.  Find the general solution using `dsolve()`.
4.  Define the initial conditions.
5.  Find the particular solution using `dsolve()` with the initial
    conditions.
6.  Display both solutions.
7.  Stop.

### Flowchart

``` text
 START
   |
 Define symbolic function y(x)
   |
 Enter homogeneous differential equation
   |
 Find general solution using dsolve()
   |
 Define initial conditions
   |
 Find particular solution using dsolve()
   |
 Display both solutions
   |
  STOP
```

------------------------------------------------------------------------

## 11. Non-Homogeneous Differential Equation

**What it does:** Finds the general and particular solutions of a
non-homogeneous differential equation using MATLAB's `dsolve()`.

### MATLAB code

``` matlab
clc;
clear;
format compact;
close all;

syms y(x)
ode = diff(y,x, 2) - 5*(diff(y,x)) + 6*y == 6*x;

% General Solution
y_gen = dsolve(ode);
disp("General Solution:");
disp(y_gen);

% Initial Conditions
cond1 = y(0) == 1;
Dy  = diff(y,x);

cond2 = Dy(0) == 0;

% Particular Solution
y_part = dsolve(ode, cond1, cond2);
disp("Particular Solution:");
disp(y_part);
```

### Algorithm

1.  Start and define symbolic function `y(x)`.
2.  Enter the non-homogeneous differential equation.
3.  Find the general solution using `dsolve()`.
4.  Define the initial conditions.
5.  Find the particular solution using `dsolve()` with the initial
    conditions.
6.  Display both solutions.
7.  Stop.

### Flowchart

``` text
 START
   |
 Define symbolic function y(x)
   |
 Enter non-homogeneous differential equation
   |
 Find general solution using dsolve()
   |
 Define initial conditions
   |
 Find particular solution using dsolve()
   |
 Display both solutions
   |
  STOP
```

------------------------------------------------------------------------

## 12. Linear (Straight-Line) Curve Fitting

**What it does:** Finds the best-fitting straight line `y = a + bx` for
given data using the least-squares method.

### MATLAB code

``` matlab
clc;
clear;
close all;

x = input('Enter x data: ');
y = input('Enter y data: ');
N = length(x);

A = [N sum(x);
     sum(x) sum(x.^2)];

B = [sum(y);
     sum(x.*y)];

phi = A\B;

a = phi(1);
b = phi(2);

fprintf("%.4f + %.4f*x\n", a, b);
y_fit = a+b*x;

err = y-y_fit;
sse = sum(err.^2);
mse = sse/N;
rmse = sqrt(mse);

plot(x,y,'o',x,y_fit,'-');
legend('Original Data','Fitted Curve');
xlabel('x');
ylabel('y');
title('Straight Line Fitting');
grid on;
```

### Algorithm

1.  Start and read x and y data.
2.  Calculate the number of data points `N`.
3.  Construct normal equation matrices `A` and `B`.
4.  Solve `phi = A\B`.
5.  Extract coefficients `a` and `b`.
6.  Calculate fitted values using `y_fit = a+b*x`.
7.  Calculate SSE, MSE, and RMSE.
8.  Plot original data and fitted line.
9.  Stop.

### Flowchart

``` text
 START
   |
 Input x and y data
   |
 Calculate N
   |
 Construct matrices A and B
   |
 Solve phi = A\B
   |
 Calculate a and b
   |
 Calculate fitted values
   |
 Calculate errors
   |
 Plot straight line
   |
  STOP
```

------------------------------------------------------------------------

## 13. Secondary Polynomial (Quadratic) Curve Fitting

**What it does:** Fits a second-degree polynomial `y = a + bx + cx²` to
the given data using the least-squares method.

### MATLAB code

``` matlab
clc;
clear;
close all;

x=[0.8 1.4 2.7 3.8 4.8 4.9];
y=[0.69 1 2.02 2.39 2.34 2.83];
N=length(y);

A=[N,sum(x),sum(x.^2);
   sum(x),sum(x.^2),sum(x.^3);
   sum(x.^2),sum(x.^3),sum(x.^4)];

B=[sum(y);
   sum(x.*y);
   sum(x.^2.*y)];

phi=A\B;
a=phi(1);
b=phi(2);
c=phi(3);

fprintf('best fit line:\n');
fprintf('y=%.4f +%.4f x+%.4f x^2\n ',a,b,c);
yFit = a + b*x + c*x.^2;

figure

scatter(x,y,'filled')

hold on
plot(x,yFit,'r-','LineWidth',2)
xlabel('x values')
ylabel('y values')
title('scatter plot with best fit line using normal equations')
legend('Data points','Best fit line')
grid on

error=y-yFit;
SSE = sum(error.^2);
MSE = mean(error.^2);
RMSE = sqrt(mean(error.^2));
fprintf('SSE = %.4f, RMSE = %.4f\n', SSE, RMSE);

SST = sum((y - mean(y)).^2);
R2 = 1 - SSE/SST;
fprintf('R^2 = %.4f\n', R2);
```

### Algorithm

1.  Start and define/input x and y data.
2.  Calculate the number of data points `N`.
3.  Construct normal equation matrices `A` and `B`.
4.  Solve `phi = A\B`.
5.  Extract coefficients `a`, `b`, and `c`.
6.  Calculate fitted values using `yFit = a+b*x+c*x.^2`.
7.  Plot original data and fitted quadratic curve.
8.  Calculate SSE, MSE, and RMSE.
9.  Calculate R².
10. Stop.

### Flowchart

``` text
 START
   |
 Input/define x and y data
   |
 Calculate N
   |
 Construct matrices A and B
   |
 Solve phi = A\B
   |
 Calculate a, b and c
   |
 Calculate fitted values
   |
 Plot quadratic curve
   |
 Calculate SSE, MSE, RMSE and R²
   |
  STOP
```

------------------------------------------------------------------------

## 14. Exponential Curve Fitting

**What it does:** Fits an exponential model `y = a*exp(b*x)` to the
given data by taking the logarithm of y and solving linear normal
equations.

### MATLAB code

``` matlab
clc;
clear;
close all;

x = input('Enter x data: ');
y = input('Enter y data: ');
N = length(x);

Y = log(y);

A = [N sum(x);
     sum(x) sum(x.^2)];

B = [sum(Y);
     sum(x.*Y)];

phi = A\B;

a = exp(phi(1));
b = phi(2);

fprintf("%.4f*exp(%.4f*x)\n", a, b);
y_fit = a*exp(b*x);

err = y-y_fit;
sse = sum(err.^2);
mse = sse/N;
rmse = sqrt(mse);

plot(x,y,'o',x,y_fit,'-');
legend('Original Data','Fitted Curve');
xlabel('x');
ylabel('y');
title('Exponential Curve Fitting');
grid on;
```

### Algorithm

1.  Start and read x and y data.
2.  Calculate the number of data points `N`.
3.  Take logarithm of y values.
4.  Construct normal equation matrices `A` and `B`.
5.  Solve `phi = A\B`.
6.  Calculate `a = exp(phi(1))` and `b = phi(2)`.
7.  Calculate fitted values using `y_fit = a*exp(b*x)`.
8.  Calculate SSE, MSE, and RMSE.
9.  Plot original data and fitted curve.
10. Stop.

### Flowchart

``` text
 START
   |
 Input x and y data
   |
 Calculate N and Y = log(y)
   |
 Construct matrices A and B
   |
 Solve phi = A\B
   |
 Calculate a and b
   |
 Calculate fitted values
   |
 Calculate errors
   |
 Plot exponential curve
   |
  STOP
```

------------------------------------------------------------------------

## 15. Power Curve Fitting

**What it does:** Fits a power model `y = a*x^b` by taking logarithms of
both x and y and solving linear normal equations.

### MATLAB code

``` matlab
clc;
clear;
close all;

x = input('Enter x values: ');
y = input('Enter y values: ');

X = log(x);
Y = log(y);

N = length(x);

A = [N sum(X);
     sum(X) sum(X.^2)];

B = [sum(Y);
     sum(X.*Y)];

phi = A\B;

a = exp(phi(1));
b = phi(2);

y_fit = a*x.^b;

disp('a ='); disp(a);
disp('b ='); disp(b);
disp('Fitted equation: y = a*x^b');
disp('Fitted values:'); disp(y_fit);

err = y-y_fit;
sse = sum(err.^2);
mse = sse/N;
rmse = sqrt(mse);

disp('SSE ='); disp(sse);
disp('MSE ='); disp(mse);
disp('RMSE ='); disp(rmse);

plot(x,y,'o',x,y_fit,'-');
legend('Original Data','Fitted Curve');
xlabel('x');
ylabel('y');
title('Power Curve Fitting');
grid on;
```

### Algorithm

1.  Start and read x and y values.
2.  Calculate `X = log(x)` and `Y = log(y)`.
3.  Calculate the number of data points `N`.
4.  Construct normal equation matrices `A` and `B`.
5.  Solve `phi = A\B`.
6.  Calculate `a = exp(phi(1))` and `b = phi(2)`.
7.  Calculate fitted values using `y_fit = a*x.^b`.
8.  Calculate SSE, MSE, and RMSE.
9.  Plot original data and fitted power curve.
10. Stop.

### Flowchart

``` text
 START
   |
 Input x and y values
   |
 Calculate X = log(x), Y = log(y)
   |
 Construct matrices A and B
   |
 Solve phi = A\B
   |
 Calculate a and b
   |
 Calculate fitted values
   |
 Calculate errors
   |
 Plot power curve
   |
  STOP
```

------------------------------------------------------------------------

## 16. Newton--Raphson Method

**What it does:** Finds a root of an equation iteratively using the
function and its derivative.

### MATLAB code

``` matlab
clc;
clear;
close all;

syms x

f_sym = input('Enter the function f(x): ');
f = matlabFunction(f_sym);

df_sym = diff(f_sym,x);
df = matlabFunction(df_sym);

tol = input('Enter tolerance: ');
maxIter = input('Enter maximum iterations: ');

fprintf('\nFinding possible interval where root lies...\n');

a = 0;
b = 100;
step = 0.5;

found = false;

for i = a:step:b-step
    if f(i)*f(i+step) < 0
        a = i;
        b = i+step;
        found = true;
        break;
    end
end

if ~found
    error('No sign-change interval found. Try changing scan range.');
end

fprintf('Root lies in interval: [%.4f, %.4f]\n',a,b);

x0 = input('Enter initial guess x0: ');

fprintf('Initial guess x0 = %.6f\n',x0);
fprintf('\nIter\t x\t\t f(x)\t\t Error\n');
fprintf('-----------------------------------------------\n');

x_old = x0;

for n = 1:maxIter

    if df(x_old) == 0
        error('Derivative is zero. Choose another initial guess.');
    end

    x_new = x_old - f(x_old)/df(x_old);
    err = abs(x_new-x_old);

    fprintf('%d\t %.8f\t %.8f\t %.8e\n',n,x_new,f(x_new),err);

    if err < tol
        fprintf('\nConverged in %d iterations\n',n);
        break;
    end

    x_old = x_new;
end

fprintf('\nApproximate root = %.10f\n',x_new);
fprintf('Final error = %.10e\n',err);
```

### Algorithm

1.  Start and define symbolic variable `x`.
2.  Read the function `f(x)`.
3.  Calculate its derivative using `diff()`.
4.  Convert the function and derivative into MATLAB functions using
    `matlabFunction()`.
5.  Read tolerance and maximum iterations.
6.  Scan the specified range for an interval with a sign change.
7.  If no interval is found, display an error and stop.
8.  Read the initial guess `x0`.
9.  Calculate the new approximation using the Newton--Raphson formula.
10. Calculate the error.
11. If the error is below tolerance, stop; otherwise update the old
    approximation and repeat.
12. Display the approximate root and final error.
13. Stop.

### Flowchart

``` text
 START
   |
 Define x, input f(x), calculate derivative
   |
 Convert f and f' to MATLAB functions
   |
 Read tolerance and maximum iterations
   |
 Scan for possible sign-change interval
   |
 Was an interval found?
   | No                    | Yes
 Display error             Read initial guess x0
 Stop                         |
                         Calculate new x using
                         x_new = x_old - f(x_old)/f'(x_old)
                               |
                         Calculate error
                               |
                         Error < tolerance?
                           | Yes        | No
                         Display       Update x_old = x_new
                         root          Repeat iteration
                           |
                          STOP
```

> The interval scan in this faculty-style program checks only from 0 to
> 100 in steps of 0.5. It is a preliminary search, not part of the
> Newton--Raphson formula itself. Choose an initial guess that is
> suitable for the root you want to find.

------------------------------------------------------------------------

# Quick Formula Revision

  Method                Main formula / idea
  --------------------- ---------------------------------------------------
  Jacobi                Uses only previous iteration values
  Gauss--Seidel         Uses latest available values
  Bisection             `c = (a+b)/2`
  Exact equation        `My = Nx`
  SVD                   `A = U*S*V'`
  PCA                   Center data, covariance, eigenvectors, projection
  Growth / Decay        `N(t) = N0*exp(k*t)`
  Newton's Cooling      `T(t) = Ts + (T0-Ts)*exp(-k*t)`
  Homogeneous ODE       RHS = 0
  Non-homogeneous ODE   RHS is non-zero
  Linear fitting        `y = a + b*x`
  Quadratic fitting     `y = a + b*x + c*x^2`
  Exponential fitting   `y = a*exp(b*x)`
  Power fitting         `y = a*x^b`
  Newton--Raphson       `x_new = x_old - f(x_old)/f'(x_old)`

# Last-Minute MATLAB Reminders

-   `A'` --- transpose of matrix A.
-   `A\B` --- solves the linear system `A*X = B`.
-   `.*` --- element-wise multiplication.
-   `.^` --- element-wise power.
-   `sum(x)` --- sum of elements.
-   `length(x)` --- number of elements in a vector.
-   `diff(f,x)` --- symbolic derivative with respect to x.
-   `dsolve(ode)` --- solves a symbolic differential equation.
-   `eig(A)` --- calculates eigenvalues and eigenvectors.
-   `matlabFunction(f)` --- converts a symbolic expression to a MATLAB
    function.
-   `log(x)` --- natural logarithm.
-   `exp(x)` --- exponential function.
-   `norm(A,inf)` --- infinity norm.
-   `plot()` --- plots lines.
-   `scatter()` --- plots data points.
-   `hold on` --- keeps the current plot while adding another plot.

**Exam tip:** Practise writing the algorithm and flowchart from memory,
then run each code once with valid inputs. For curve fitting, remember
the four model equations and how logarithms are used in exponential and
power fitting.
