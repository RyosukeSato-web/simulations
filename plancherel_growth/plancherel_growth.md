``` python
# %% 
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.animation as animation

# %%
mins = np.array([-3, -1, 2, 5])
maxs = np.array([-2, 1, 4])

def x_axis_func (mins, maxs):
    return np.sort(np.append(mins, maxs))
x_axis = x_axis_func(mins, maxs)
# print(x_axis)

def height_array (mins, maxs):
    if(len(maxs)==0):
        return np.array([np.abs(mins[0])])
    else:
        # print(len(mins), "aaa")
        assert len(mins) >=2, f"{mins} ?"
        h = height_array(mins[:-1], maxs[:-1])
        b = h[-1] + maxs[-1] - mins[-2]
        tail = np.array([b, b - mins[-1] + maxs[-1]])
        return np.append(h, tail)

height = height_array(mins, maxs)
# print(height)

# %%
plt.axes().set_aspect('equal')
plt.plot(x_axis, height)
plt.plot([-6, 0, 6], [6, 0, 6])

# %%
def add_point(mins, maxs):
    x_min = mins[0]
    x_max = mins[-1]

    while x_min != x_max:
        u=np.random.uniform(x_min, x_max)
        i_x = np.count_nonzero(mins < u)
        i_y = np.count_nonzero(maxs < u)
        if(i_x == i_y):
            x_min = mins[i_x]
        else:
            x_max = mins[i_x-1]

    return x_max

print(add_point(mins, maxs))

# %%
def plancherel_growth(mins, maxs):
    x_add = add_point(mins, maxs)
    i = np.count_nonzero(maxs < x_add)
    # print(i)
    if(mins[i]-1 in maxs):
        new_maxs = np.sort(np.append(np.delete(maxs, i-1), mins[i]))
        new_mins = np.sort(np.append(np.delete(mins, i), mins[i]+1))
    elif(mins[i]+1 in maxs):
        new_maxs = np.sort(np.append(np.delete(maxs, i), mins[i]))
        new_mins = np.sort(np.append(np.delete(mins, i), mins[i]-1))
    else:
        new_maxs = np.sort(np.append(maxs, mins[i]))
        new_mins = np.sort(np.append(np.delete(mins, i), np.array([mins[i]-1, mins[i]+1])))
    return new_mins, new_maxs

new_diag = plancherel_growth(mins, maxs)
# print(type(new_diag[0]))

new_x_axis = x_axis_func(new_diag[0], new_diag[1])
new_height = height_array(new_diag[0], new_diag[1])

plt.axes().set_aspect('equal')
plt.plot(x_axis, height, color="red")
plt.plot([-6, 0, 6], [6, 0, 6])
plt.plot(new_x_axis, new_height, color="blue")

# %%
MINS_ORIGIN = np.array([-1, 1])
MAXS_ORIGIN = np.array([0])

SIM_TIME = 1000

def limit_shape_sim (time, mins_origin, maxs_origin):
    new_diag = (mins_origin, maxs_origin)
    for t in range(time):
        new_diag = plancherel_growth(new_diag[0], new_diag[1])
    return new_diag
    
result_diag = limit_shape_sim(SIM_TIME, MINS_ORIGIN, MAXS_ORIGIN)

x_axis = x_axis_func(result_diag[0], result_diag[1]) * (1.0/np.sqrt(SIM_TIME))
height = height_array(result_diag[0], result_diag[1]) * (1.0/np.sqrt(SIM_TIME))

x_sample = np.linspace(-2, 2, 100)
vkls = (2.0/np.pi)*(x_sample*np.arcsin(x_sample/2)+np.sqrt(4-x_sample**2))


plt.axes().set_aspect('equal')
plt.plot(x_axis, height, color="blue")
plt.plot([-2.5, 0, 2.5], [2.5, 0, 2.5], color="black")
plt.plot(x_sample, vkls, color="red")

# %%
fig = plt.figure(figsize=(8, 5))
# ax = fig.add_subplot(111)

x_sample = np.linspace(-2, 2, 100)
vkls = (2.0/np.pi)*(x_sample*np.arcsin(x_sample/2)+np.sqrt(4-x_sample**2))

SIM_TIME = 500
MINS_ORIGIN = np.array([-1, 1])
MAXS_ORIGIN = np.array([0])

diag = (MINS_ORIGIN, MAXS_ORIGIN)

images = []
plt.axes().set_aspect('equal')

for t in range(SIM_TIME):
    x_axis = x_axis_func(diag[0], diag[1])
    height = height_array(diag[0], diag[1])

    x_axis_scal = x_axis*(1.0/np.sqrt(t))
    height_scal = height*(1.0/np.sqrt(t))

    image1 = plt.plot(x_axis_scal, height_scal, color="blue")
    image2 = plt.plot([-2.5, 0, 2.5], [2.5, 0, 2.5], color="black")
    image3 = plt.plot(x_sample, vkls, color="red")
    images.append(image1 + image2 + image3)

    diag = plancherel_growth(diag[0], diag[1])

anime = animation.ArtistAnimation(fig, images, interval=40, blit=True, repeat_delay=0)
anime.save("plancherel_growth.gif", writer="pillow")
plt.show()

# %%

```